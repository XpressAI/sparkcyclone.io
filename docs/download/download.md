---
sidebar_position: 1
---
# Downloads

[Spark Cyclone](https://github.com/XpressAI/SparkCyclone) provides a set of plugins for Apache Spark that leverage VEs to accelerate Dataframe and SQL processing.

See the getting-started guide for more details.

## Release v1.0.1

The release can be downloaded on the [Spark Cyclone 1.0.1 Release](https://github.com/XpressAI/SparkCyclone/releases/tag/v1.0.1) page on Github.

### Installation

#### Global Installation method

- Copy the jar file to /opt/cyclone on all machines 
- Specify the jar file path in the `--jars` option to `spark-submit` 
- Specify the jar file path as a `--conf` option inside `spark.executor.extraClassPath`

#### Spark Archive method

- Include the jar file in your job's spark archive zip file and specify as `spark.yarn.archive`

### Hardware Requirements:

The plugin is tested on the following architectures:

- SX-Aurora TSUBASA Vector Engine Type 20B
- SX-Aurora TSUBASA Vector Engine Type 10AE
- SX-Aurora TSUBASA Vector Engine Type 10B
- SX-Aurora TSUBASA Vector Engine Type 10BE
- SX-Aurora TSUBASA Vector Engine Type 10CE


### Software Requirements:

- NEC VE SDK 3.4.0. (Installed with nec-sdk-devel group)
- Apache Spark 3.1.0, 3.1.1, 3.1.2, 3.1.3
- Apache Hadoop 3.3.0
- Python 3.6+
- Scala 2.12
- Java 8


For officially supported OSes and kernel versions, please visit [NEC's page.](http://www.support.nec.co.jp/en/View.aspx?id=4140100078)


## Release Notes

New functionality and performance improvements for this release include:

- Updated defaults to allow more Spark phases to run on the VE.
- Replace std::max and std::min with MIN/MAX macros improving vectorization.
- Code generation cleanups around sort.
- Update internal Frovedis code to the latest master.

For a detailed list of changes, please refer to the release page.
