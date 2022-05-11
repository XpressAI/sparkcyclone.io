---
sidebar_position: 4
---

# Usage

The Cyclone plug-in offers two ways to run operations on the NEC SX-Aurora
Vector Engine.  You can use either Spark SQL or the new VeRDD API.  The Spark
SQL API is implemented as a plug-in into the Catalyst SQL engine in Spark and
therefore typically does not require any changes to applications to be used.
Simply specifying the plug-in when running spark-submit is sufficient.  The
VeRDD API is currently more limited but offers great performance at the cost
of requiring code to depend on the Spark Cyclone plug-in and convert RDDs to
VeRDDs.

## Enabling the plug-in with Spark SQL.

The minimum configuration required to start a job that uses Cyclone after 
completing all the setup steps in the previous sections is:

```bash
 $SPARK_HOME/bin/spark-submit --master yarn \
        --num-executors=8 --executor-cores=1 --executor-memory=4G \
        --jars /opt/cyclone/spark-cyclone-sql-plugin.jar \
        --conf spark.executor.extraClassPath=/opt/cyclone/spark-cyclone-sql-plugin.jar \
        --conf spark.plugins=com.nec.spark.AuroraSqlPlugin \
        --conf spark.executorEnv.VE_OMP_NUM_THREADS=1 \
        --conf spark.com.nec.spark.kernel.directory=/opt/spark/work/cyclone \
        your_job_jar_file.jar
```

### --master yarn

Spark Cyclone uses a combination of AVEO and Frovedis for offloading 
computations onto the Vector Engine.  As Frovedis is implemented with MPI its
algorithms require one process per Vector Engine core. Therefore it is 
necessary to use YARN (or Kubernetes) to launch 8 processes per Vector Engine
card.

### --num-executors=8 --executor-cores=1

The num-executors option should be set to 8 x the total number of Vector Engine
cards on the machine.  These cores will handle communication with the VE for
offloading.  The AVEO API also requires that a single thread operates on its
data structures, so it is also necessary to specify --executor-cores=1 to 
prevent extra threads from being used.

### --jars and extraClassPath

The --jars option specifies extra jars to be put into the driver class path. To
use the plugin it is necessary for the driver to have this jar loaded, and the
NCC compiler to be installed on the driver system.  The driver does not utilize
the Vector Engine and so it does not need to run on a system with Vector Engines
installed.

Executors also use Spark Cyclone APIs to offload computations so they also 
require that the Spark Cyclone jar is on the class path.

### --conf spark.plugins=com.nec.spark.AuroraSqlPlugin

To enable the plugin it is necessary to add `com.nec.spark.AuroraSqlPlugin`
to the `spark.plugins` conf variable in addition to any other plugins you
might want to use.

### --conf spark.executorEnv.VE_OMP_NUM_THREADS=1

As currently Frovedis uses single threaded algorithms, it is necessary to 
also limit the number of OpenMP threads used by the AVEO library to 1 (as
we are using 8 separate processes.)

### --conf spark.com.nec.spark.kernel.directory=/opt/spark/work/cyclone

Spark Cyclone works be converting the Spark SQL queries to C++ code, compiling
them and executing them with on the Vector Engine with AVEO.  Compilation of
the C++ kernels can take a significant amount of time. (Currently up to about
35 seconds.)  By specifying a `kernel.directory` Spark Cyclone will save the
source code and compiled `.so` files in this directory and reuse the compiled
version if it's compatible which saves 35 seconds on the second query execution.

### Other options

By default the plugin will use all available Vector Engines on the system.  To
share Vector Engines among many jobs it's necessary to specify the following
configurations to leverage YARN for resource management.

    --conf spark.executor.resource.ve.amount=1 \
    --conf spark.executor.resource.ve.discoveryScript=/opt/spark/getVEsResources.py \
