# Findings

## F-01: Default controller runtime inherits the Apache Spark image

**Observation**

The Spark Operator image is built on `docker.io/apache/spark:4.0.4`, pinned by digest.

The default SparkApplication submission path invokes `${SPARK_HOME}/bin/spark-submit` from that runtime image.

**Evidence**

- `Dockerfile`
- `internal/controller/sparkapplication/submission.go`
- `docs/website/user-guide/building-custom-images.md`
- repository revision:
  `628a90f518e14589e342d002404810d4d5d92ce1`

**Security implication**

Vulnerability findings for the complete controller image can include components inherited from the selected Apache Spark runtime, not only the Spark Operator Go binary and its direct dependencies.

**Remediation boundary**

The inherited dependency surface has been partially characterized. Remediation ownership for individual findings is still under investigation.

**Status**

Verified

## F-02: The controller inherits a broad Spark/JVM dependency surface

**Observation**

The Apache Spark 4.0.4 base image contains 276 JAR files as well as a JVM, Hadoop client libraries, Python, and multiple JVM dependency families including Netty, Jackson, Arrow, Parquet, and ORC.

**Evidence**

Runtime inspection of:

`docker.io/apache/spark:4.0.4@sha256:94ad730f7510002d8a1615de269f27cdeca4d4eef51657384db3fa9246b5a4d8`

Examples include:

- `hadoop-client-api-3.4.1.jar`
- `hadoop-client-runtime-3.4.1.jar`
- `netty-handler-4.1.118.Final.jar`
- `jackson-databind-2.18.6.jar`
- `arrow-vector-18.1.0.jar`
- `parquet-hadoop-1.15.2.jar`
- `orc-core-2.1.4-shaded-protobuf.jar`

**Security implication**

A vulnerability report for the complete controller image can include dependencies inherited from the Spark distribution rather than only dependencies maintained directly by Spark Operator.

**Remediation boundary**

The exact remediation path depends on whether the affected component is owned by Spark Operator, inherited from the base operating system, or bundled with the Apache Spark distribution.

**Status**

Verified

## F-03: Spark Operator does not modify the Spark JAR set during image construction

**Observation**

The SHA-256 hashes of all JAR files under `$SPARK_HOME/jars` were compared between the pinned Apache Spark 4.0.4 base image and a Spark Operator image built from the audited repository revision.

No differences were found.

**Evidence**

- 276 JARs were identified in the Apache Spark base image.
- Both base and final-image SHA-256 manifests contained 276 JARs.
- `diff` produced no differences.

**Security implication**

JAR-level vulnerability findings in the final controller image can be traced to the selected Spark distribution when the affected JAR is present unchanged in both images.

**Remediation boundary**

Provenance is established, but the appropriate remediation path for individual Spark dependencies still requires compatibility and upstream-support analysis.

**Status**

Verified

## H-01: Repository-local scanning may not cover the full controller image

The repository's visible OSV workflow scans source dependency manifests. `SECURITY.md` separately states that container images are scanned.

No repository-local image-scanning workflow has been identified yet. Image scanning may be provided elsewhere in the project's release or organization-level infrastructure, so this remains under investigation.

**Status**

Needs further investigation.
