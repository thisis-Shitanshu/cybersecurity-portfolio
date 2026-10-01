# Kubeflow Spark Operator Supply-Chain Security Analysis

## Overview

This project investigates the security boundary of the Kubeflow Spark Operator controller image.

The controller is built on an Apache Spark runtime image, which means vulnerability scans can report issues from several different sources:

- Spark Operator
- Apache Spark
- Hadoop
- JVM dependencies
- the base operating system

The goal was to identify where those findings originate and understand what safe remediation would require.

## Upstream context

This work supports Kubeflow Spark Operator issue
[#3143](https://github.com/kubeflow/spark-operator/issues/3143).

The issue asks for security guidance around CVEs inherited from the Spark base image.

## What I investigated

- how the Spark Operator controller image is built
- why the controller contains an Apache Spark runtime
- which dependencies are inherited from Spark
- how vulnerability findings differ between the Spark base image and Spark Operator image
- how shaded dependencies inside Hadoop affect remediation
- whether scanner-reported fixed versions can be applied safely
- how dependency changes should be validated

## Key findings

The investigation established that:

1. The Spark Operator controller inherits a large JVM dependency surface from Apache Spark.

2. The Spark Operator build does not modify the JAR files already present in the Spark base image.

3. In a controlled paired scan, all OS and Java findings in the controller image were already present in the Spark base image. The additional findings were associated with the Spark Operator Go binary.

4. Some vulnerable dependencies are embedded inside shaded Hadoop artifacts rather than existing only as standalone JAR files.

5. A scanner-reported fixed version is not necessarily a safe drop-in replacement. Updating a shaded dependency may require coordinated dependency changes, build-tool changes, rebuilding the parent artifact, runtime testing, and rescanning.

## Remediation experiment

A controlled experiment rebuilt Hadoop's `hadoop-client-runtime-3.4.1.jar` with Jackson `2.18.11`.

The experiment uncovered two compatibility problems:

- Hadoop's Maven Shade Plugin `3.4.1` could not process a Java 21 multi-release class from the newer Jackson version.
- upgrading Jackson changed its JAXB dependency, which interacted with an existing Hadoop dependency exclusion and caused runtime class-resolution failures.

After updating the Shade Plugin and correcting the JAXB dependency path, the rebuilt artifact:

- built successfully
- passed a targeted JAXB linkage test
- worked inside the exact Spark 4.0.4 runtime
- completed a local Spark job
- reduced the Java vulnerability findings attributed to the Hadoop runtime without adding new findings in the recorded scan snapshot

This was a controlled prototype, not a production patch.

## Documents

- [Architecture](architecture.md)
- [Findings](findings.md)
- [Outcome](outcome.md)
- [References](references.md)

## Skills demonstrated

- software supply-chain security
- container and OCI image analysis
- vulnerability scanning
- dependency provenance analysis
- Maven and Java dependency analysis
- shaded dependency investigation
- Kubernetes operator architecture
- runtime compatibility testing
- security documentation
- open-source collaboration

## Upstream work

Issue:
[#3143](https://github.com/kubeflow/spark-operator/issues/3143)

Pull request:
[#3214](https://github.com/kubeflow/spark-operator/pull/3214)
