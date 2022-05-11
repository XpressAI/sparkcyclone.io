---
sidebar_position: 2
---

# Building CycloneML

## Building Spark MLLib

The CycloneML component is a fork of Apache Spark that has MLLib modified to run using Frovedis.  To build it follow
these steps:

```shell

git clone https://github.com/XpressAI/CycloneML.git
cd CycloneML/mllib
git checkout aurora
mvn clean install -Dmaven.test.skip
```

## Making a Spark Archive

The best way to include CycloneML into your application is to add it to a custom Spark Archive and specify the
`spark.yarn.archive` configuration when running spark-submit.

To create an archive you can execute the following commands (Assuming you have built Spark Cyclone):

```shell
mkdir spark-cycloneml
cp -r $SPARK_HOME/jars/* spark-cycloneml/
cp target/scala-2.12/*.jar spark-cycloneml/
cp /opt/cyclone/${USER}/spark-cyclone-sql-plugin.jar spark-cycloneml/
zip -r spark-cycloneml.zip spark-cycloneml
```

Then you can specify the location of the zip file as a `spark.yarn.archive`

```shell
$SPARK_HOME/bin/spark-submit \
	--master yarn \
	--num-executors=1--executor-cores=8 --executor-memory=7G \
	--name job \
	--conf spark.plugins=com.nec.spark.AuroraSqlPlugin \
	--conf spark.executor.resource.ve.amount=1 \
	--conf spark.executor.resource.ve.discoveryScript=/opt/spark/getVEsResources.py \
	--conf spark.yarn.archive=./spark-cycloneml.zip
	job.py
```
