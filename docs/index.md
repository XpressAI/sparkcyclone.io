---
sidebar_position: 0
---

# Overview

The Spark Cyclone plugin accelerates the performance of Spark by using the SX-Aurora Vector Engine (VE), making it an
exiting new option for Big Data Analytics use cases. The plugin enables developers to accelerate Spark jobs by 
generating C++ and executing it on th VE hardware. We believe that the architecture of the VE makes it better suited for
 Big Data workloads than other accelerators in the market as it offers support for the typical data types used in Spark
jobs (such as Long and Double) and has much larger memory capacity and higher memory bandwidth than most accelerators
available on the market.

In these docs you'll fine a quick start guide on how to start using Spark on the VE and sample commands, as well as how
to install Spark Cyclone plugin, and how to build it from scratch. We also introduce the concepts related to the
implementation of Spark and Kubernetes.

Cyclone currently offers three ways to accelerate Spark on the Vector Engine.  Spark SQL, VERDD, and MLLib.

## Spark SQL

Spark SQL is built with an extensibility system that we leverage to accelerate SQL queries with the Vector Engine.  With
this API other than the configuration of spark-submit, no other code changes are necessary.  It will run completely in
the background and run queries on the VE if the datatypes are supported.

## VERDD

The VERDD API gives you more direct access to the Vector Engine.  The API provides Scala macros that can be used to
convert normal Scala code into expressions which get transpiled to C++ and executed on the VE.  With this API it is 
necessary to change your code to explicitly use the VERDD class and compile it so that the C++ code can be generated.
There are some limitations to the expressions you can use, but the performance gains with of the VERDD API is very
significant.

## MLLib

CycloneML is a fork of MLLib that accelerates many of the ML algorithms with either the Vector Engine or CPU.  In a 
similar spirit to the Spark SQL support, you can take advantage of the VE without rewriting your code, but you must
add CycloneML into your job classpath.
