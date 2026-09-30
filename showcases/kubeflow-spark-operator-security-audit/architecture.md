# Architecture

This document will record the verified image and runtime architecture discovered during the investigation.

## Questions to verify

- What image is used as the Spark Operator controller base?
- Which Spark components are present in that image?
- Which Hadoop/JVM dependencies are present?
- What does the Spark Operator build add?
- Which images are used separately by Spark driver and executor Pods?

## Verified architecture

At repository revision `628a90f518e14589e342d002404810d4d5d92ce1`:

- The Spark Operator controller image uses `docker.io/apache/spark:4.0.4` as its runtime base image, pinned by digest.
- The Spark Operator executable is built separately with Go and copied into the Spark-based runtime image.
- The runtime image additionally installs `catatonit`.
- The default SparkApplication submission implementation uses `SparkSubmitter`.
- `SparkSubmitter` invokes `${SPARK_HOME}/bin/spark-submit` as a subprocess.
- The local `spark-submit` path starts a JVM inside the controller Pod.
- An experimental `RestSubmitter` feature gate exists as an alternative submission strategy. It is alpha and disabled by default at this revision.

### SparkApplication submission

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
Driver Pod -> Executor Pods

```

The repository also contains an experimental REST submission path behind the RestSubmitter alpha feature gate. When enabled, the controller sends the generated Spark submission arguments to an external submitter service instead of spawning spark-submit locally.

The controller-side REST path sends generated Spark submission arguments to an external submitter service instead of spawning `spark-submit` locally. The submitter-service implementation and its runtime dependency surface are outside this repository and are not part of the current image audit.

### Runtime dependency composition

At the audited revision, the controller runtime inherits the `apache/spark:4.0.4` image.

Runtime inspection identified:

- Ubuntu 22.04.5 LTS
- Eclipse Temurin OpenJDK 17.0.19+10
- Apache Spark 4.0.4 built with Hadoop 3 support
- 276 JAR files under `$SPARK_HOME/jars`
- Hadoop client 3.4.1
- Arrow 18.1.0
- Jackson 2.18.6
- Netty 4.1.118.Final
- Parquet 1.15.2
- ORC 2.1.4
- Python 3.10.12

The Spark Operator build adds comparatively few runtime components on top of this base:

- the statically linked `spark-operator` executable, approximately 48 MB
- `entrypoint.sh`
- `catatonit` 0.1.7-1
- directories and permissions required by the webhook/controller

A package-level comparison between the Apache Spark base image and the locally built Spark Operator image identified `catatonit` as the only additional Debian package.

The Spark Operator executable was verified as a statically linked ELF binary. `ldd` reported that it is not dynamically linked.

**Further dependency composition is still under investigation.**

## Security scanning

- How are Spark Operator source dependencies scanned?
- Where are published controller images scanned?
- Does image scanning include inherited Spark/JVM dependencies?
- Are scan results visible to users?
- Does the current implementation match the guarantee in `SECURITY.md`?
