---
sidebar_position: 1
---

# VeRDD API

The VeRDD API is the new API available to offload operations onto the Vector
Engine.  As many of the complexities of SQL are skipped with this more direct
API it is possible to be more efficient with Vector Engine usage.  The VeRDD
API works by using Scala Macros to translate Scala expressions into C++ which
is then compiled and executed on the Vector Engine in much of the same way as
the Spark SQL API.

### Creating a VeRDD

Creating a VeRDD is as simple as adding .toVeRDD to the input RDD after importing
the implicit classes provided.  An example is shown below.

```scala
import com.nec.ve.VeRDD._
import org.apache.spark.{SparkConf, SparkContext}

val conf = new SparkConf().setAppName("VeRDD Test")
val sc = new SparkContext(conf)

val rdd: RDD[Int] = sc.parallelize(Seq(1, 2, 3, 4, 5, 6, 7, 8))
val ve: VeRDD[Int] = rdd.toVeRDD
```

Calling toVeRDD causes the RDD to get evaluated and loaded into the Vector
Engine's RAM.

There is also a `veParallelize` extension method on SparkContext that is
able to create a VeRDD from a Range.

### Executing map/filter/reduce

Using the power of Scala's macros it is possible to use the same RDD APIs
such as map, filter and reduce.

Below is an example of the same operations on bother an RDD and a VeRDD:

```scala
import com.nec.ve.VeRDD._
import org.apache.spark.{SparkConf, SparkContext}

val conf = new SparkConf().setAppName("VeRDD Test")
val sc = new SparkContext(conf)

val numbers = (1L to (500 * 1000000))
val rdd = sc.parallelize(numbers).cache()
rdd
  .filter((a: Long) => a % 3 == 0 && a % 5 == 0 && a % 15 == 0)
  .map((a: Long) => ((2 * a) + 12) - (a % 15))
  .reduce((a: Long, b: Long) => a + b)

val verdd = rdd.toVeRDD // Or sc.veParallelize(numbers)
verdd
  .filter((a: Long) => a % 3 == 0 && a % 5 == 0 && a % 15 == 0)
  .map((a: Long) => ((2 * a) + 12) - (a % 15))
  .reduce((a: Long, b: Long) => a + b)
```

As you can see, when using this API it is possible to run code on the VE with
no difference other than the call to `toVeRDD`.

Not all APIs are supported to run on the VE.  If a method that is not
implemented is used, VeRDD will automatically perform the current VeRDD
operation DAG on the VE and then convert the result into a normal `RDD[T]`
where the rest of the calls will execute as they normally do.

If you'd like to convert back into a `VeRDD[T]` from a `RDD[T]` you can add
a call to `toVeRDD`

### Limitations

There are limitations to the macro-based API that should be understood to
fully take advantage of the VeRDD API.

1) The compiler needs to know it is working with a VeRDD[T]

To generate C++ code form the defined lambda functions it is necessary to
involve a macro that rewrites the invocation of filter, map, reduce, etc.
into an `Expr`.  Due to limitations of the compiler, it is not possible for
VeRDD to override the definitions of many operations such as`map` or `groupBy`
Therefore to run the macro properly the compiler must know it is working with
a VeRDD[T] instance.  Scala's type inference is usually sufficient so it is
not necessary in general to specify the type `VeRDD[T]` explicitly.  Unless, for
example, you have a function like:

```scala
def foo(rdd: RDD[(Long, Long)]): RDD[(Long, Double)] = {
  val filtered = rdd.filter((t: (Long, Long)) => t._1 < 10)
  rdd.map((t: (Long, Long)) => (t._1, t._2 / 100.0))
}
```

In this case even if you pass a VeRDD into this function (which will work as
`VeRDD[T]` does implements `RDD[T]`) it will run the `filter` and `map` methods
that are defined on RDD and not the ones defined in VeRDD.  In this case since
and `RDD` API is being called, the `VeRDD` passed in will copy the data out of
the VE and continue as a normal RDD.  To run this on the VE it is necessary to
overload or redefine this function as taking and returning a `VeRDD` like so.

```scala
def foo(rdd: VeRDD[(Long, Long)]): VeRDD[(Long, Double)] = {
  val filtered = rdd.filter((t: (Long, Long)) => t._1 < 10)
  rdd.map((t: (Long, Long)) => (t._1, t._2 / 100.0))
}
```

2) Lambdas should have types for their parameters

While it is possible to infer the type of the lambda function
parameters based on the type of the `RDD`, it is unfortunately not possible
for the compiler to pass that information into the `Expr`.  To transpile the
expression into C++ it is necessary for the lambda to be as stand alone
as possible, and therefore necessary to define the types for the inputs.

3) Lambdas currently only support a limited number of types

It is currently possible to use `Int`, `Long`, `Double`, `Float` and `Instant` inside
of lambda function parameters.  It is also necessary that there's no implicit
conversions used inside the lambda function, as that is not C++ compatible.

4) The lambda function must contain the entire code

Calling functions defined elsewhere will get translated in to function calls
in C++.  With the exception of some simple numerical functions, the function
will not in general be available on the VE.  It is not currently possible to
have scala inline the function when reifying it into an `Expr`, but it might
become possible in the future.

5) Custom classes or case classes are not supported

Currently it is necessary to use Tuples when dealing with groups of values
as it is not currently possible to inline the definition of the case class.
This limitation might be lifted in the future.

### Explicit API

The RDD compatible methods of the VeRDD API leverage macros to rewrite the
method call.  With `map` for example, it is rewritten into `vemap`.  There
are functions for `vefilter`, `vegroupBy`, `vereduce`, `vesortBy` and more.

The example above can be converted to use the `ve` methods as follows:

```scala

val verdd = sc.veParallelize(numbers)
verdd
  .vefilter(reify { (a: Long) => a % 3 == 0 && a % 5 == 0 && a % 15 == 0 })
  .vemap(reify { (a: Long) => ((2 * a) + 12) - (a % 15) })
  .vereduce(reify { (a: Long, b: Long) => a + b })
```

In general there's no advantage to using the ve versions of the API other
than to be more explicit that it the code will run on the VE.
