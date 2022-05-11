---
sidebar_position: 1
---

# CycloneML: MLLib on VE

## Configuration

The CycloneML component is a fork of MLLib that uses Frovedis to accelerate many MLLib algorithms on the Vector Engine.
In a similar spirit to Spark Cyclone's Spark SQL support, it nearly requires no code changes to work, you only need to 
include our JAR file into your job's class path and call the initialization code described below.  

Due to some technical limitations, MLLib is implemented in a different way than the rest of Spark Cyclone.  For best
results, we recommend having separate jobs for SparkSQL and MLLib.

CycloneML is implemented with the JavaCPP Presets for Frovedis.  This JavaCPP preset is available at:

- https://github.com/bytedeco/javacpp-presets/tree/aurora/frovedis

The presets automatically bundle all the contents of the Frovedis RPM inside the JAR files that are automatically 
downloaded as part of the build. Moreover, the equivalent of the veenv.sh or x86env.sh Bash scripts is performed inside
the JVM automatically at initialization time inside user applications. The three steps that users still need to 
perform are to:

1.	Add a single dependency to their pom.xml, build.gradle, build.sbt, etc files:

For the VE version of Frovedis:

```xml
    <dependency>
        <groupId>org.bytedeco</groupId>
        <artifactId>frovedis-platform-ve</artifactId>
        <version>1.0.0-1.5.7-SNAPSHOT</version>
    </dependency>
```

Or alternatively this for the x86 version of Frovedis:

```xml
    <dependency>
        <groupId>org.bytedeco</groupId>
        <artifactId>frovedis-platform</artifactId>
        <version>1.0.0-1.5.7-SNAPSHOT</version>
    </dependency>
```

2. Call `frovedis_server.initialize(...)` and `frovedis_server.shut_down()` in the `main()` method of the job.

3. Include the CycloneML JAR file into the driver and executor classpath. (Either by specifying it with the 
   extraClassPath configuration or by bundling it in the Spark archive.)

The CycloneML code is available at:

- https://github.com/XpressAI/CycloneML 