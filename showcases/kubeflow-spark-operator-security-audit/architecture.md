# Architecture

The following architecture was verified at Spark Operator repository revision:

`628a90f518e14589e342d002404810d4d5d92ce1`

## Controller image

The Spark Operator controller image uses:

`docker.io/apache/spark:4.0.4`

as its runtime base image.

The Spark Operator executable is built separately with Go and copied into the Spark runtime image.

The final image also adds:

- the `spark-operator` executable
- `entrypoint.sh`
- `catatonit`
- controller and webhook directories and permissions

A package comparison identified `catatonit` as the only additional Debian package added on top of the Spark base image.

The `spark-operator` executable is a statically linked ELF binary.

## SparkApplication submission

The default submission path is:

```text
SparkApplication
      |
      v
Spark Operator controller
      |
      v
buildSparkSubmitArgs()
      |
      v
SparkSubmitter
      |
      v
${SPARK_HOME}/bin/spark-submit
      |
      v
     JVM
      |
      v
Kubernetes API
      |
      v
Driver Pod
      |
      v
Executor Pods
```

The controller therefore requires the local Spark runtime for the default submission path.

An experimental `RestSubmitter` feature gate also exists. It is alpha and disabled by default at the audited revision.

When enabled, the controller sends Spark submission arguments to an external submitter service instead of launching `spark-submit` locally.

That external service is outside the scope of this image audit.

## Runtime dependency surface

Inspection of the Spark runtime identified:

- Ubuntu 22.04.5 LTS
- Eclipse Temurin OpenJDK 17.0.19+10
- Apache Spark 4.0.4
- Hadoop client 3.4.1
- Python 3.10.12
- 276 JAR files under `$SPARK_HOME/jars`

Major dependency families included:

- Jackson
- Netty
- Arrow
- Parquet
- ORC
- Hadoop
