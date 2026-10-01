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

## F-04: Paired image scanning attributes the vulnerability delta to the Spark Operator Go binary

**Observation**

The pinned Apache Spark base image and the locally built Spark Operator controller image were scanned using the same Trivy version, vulnerability database snapshots, Java database snapshot, scan options, and Docker-archive input method.

The scan produced:

| Target             | Base image | Operator image |
| ------------------ | ---------- | -------------- |
| Ubuntu findings    | 4,670      | 4,670          |
| Java/JAR findings  | 173        | 173            |
| Go binary findings | 0          | 19             |

Using package type, package name, installed version, and vulnerability ID as the finding identity:

- base unique findings: 4,805
- operator unique findings: 4,824
- shared findings: 4,805
- base-only findings: 0
- operator-only findings: 19

All 19 operator-only findings were associated with the `usr/bin/spark-operator` Go binary.

**Evidence**

Scanner:

`Trivy 0.74.0`

Scanner image:

`aquasec/trivy@sha256:62b1e65e8869bc4b4c6aa4fa2b21595256c7c2f6018a9d9ad61caf87187c1969`

The vulnerability and Java databases were downloaded before the paired scan and updates were disabled during both scans.

Database snapshots:

- Trivy vulnerability DB SHA-256:
  `8612d29f9995a4897a3639b8469c62f92021ed692d0cb399090f09e18ce21f10`
- Trivy Java DB SHA-256:
  `37418de6e936d802e94712d3d3e6e41f189bdfffd0fa23ea88b383dab4543137`

The paired OS finding manifests contained 4,670 entries each and had the same SHA-256 digest:

`01b6eb4f22dfaa6917ece96b0c22a5c8e587d8bd906d192d06819e728fa1125c`

The paired Java finding manifests contained 173 entries each and had the same SHA-256 digest:

`0c3a44a830f638a1dba9fd805763fde14feb8585f555735a07ce48150ed24054`

**Security implication**

The paired scan distinguishes vulnerabilities inherited from the selected Spark base image from additional findings associated with components added by the Spark Operator image build.

At this audited revision and database snapshot, all OS and Java/JAR findings reported for the controller image were already present in the pinned Apache Spark base image. The additional findings were associated with the Spark Operator Go binary.

**Limitations**

Scanner findings do not by themselves establish exploitability or runtime reachability. Results are specific to the scanner and vulnerability database snapshots used for this audit.

**Status**

Verified

## F-05: Vulnerable JVM components may be shaded or present in multiple Spark distribution artifacts

**Observation**

The Java vulnerability scan identified some dependency versions at more than one physical location in the Spark distribution.

For example, Trivy detected `io.netty:netty-handler` version `4.1.118.Final` both in:

- `opt/spark/jars/netty-handler-4.1.118.Final.jar`
- `opt/spark/jars/connect-repl/spark-connect-client-jvm_2.13-4.0.4.jar`

Trivy also detected `com.fasterxml.jackson.core:jackson-databind` version `2.12.7.1` inside:

- `opt/spark/jars/hadoop-client-runtime-3.4.1.jar`

while Spark separately contains:

- `opt/spark/jars/jackson-databind-2.18.6.jar`

Inspection of `hadoop-client-runtime-3.4.1.jar` confirmed that Jackson classes are physically included under the relocated namespace:

`org/apache/hadoop/shaded/com/fasterxml/jackson/...`

This establishes that the Hadoop client runtime contains a shaded copy of Jackson rather than only referring to the standalone Spark Jackson JAR.

The Java scan contained:

- 173 vulnerability occurrences
- 135 unique package/version/vulnerability tuples
- 99 distinct vulnerability or advisory IDs
- 44 distinct package/version pairs

**Security implication**

Replacing a standalone dependency JAR may not eliminate all instances of an affected component from the Spark distribution.

A component may also exist inside a shaded or bundled parent artifact. For example, replacing Spark's standalone Jackson JAR would not automatically replace the Jackson implementation shaded into `hadoop-client-runtime-3.4.1.jar`.

Similarly, the same Netty component can be represented both by a standalone Netty JAR and inside another Spark artifact.

**Remediation boundary**

Safely changing a shaded dependency may require rebuilding the parent artifact rather than replacing a single file. Compatibility analysis must therefore consider the parent project, dependency versions, relocation rules, build process, and regression-test requirements.

**Status**

Verified

## H-01: Repository-local scanning may not cover the full controller image

The repository's visible OSV workflow scans source dependency manifests. `SECURITY.md` separately states that container images are scanned.

No repository-local image-scanning workflow has been identified yet. Image scanning may be provided elsewhere in the project's release or organization-level infrastructure, so this remains under investigation.

**Status**

Needs further investigation.
