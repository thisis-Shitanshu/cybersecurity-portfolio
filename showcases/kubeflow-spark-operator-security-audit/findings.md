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

The inherited dependency surface has been partially characterized. F-05 demonstrates that remediation of at least one shaded dependency requires changes and compatibility validation at the parent Hadoop/Spark artifact boundary rather than a simple controller-image patch.

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

Provenance is established. F-05 demonstrates that remediation of an inherited shaded dependency can require rebuilding and validating its parent artifact rather than replacing an individual JAR in the Spark Operator image.

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

- base unique finding tuples: 4,805
- operator unique finding tuples: 4,824
- shared finding tuples: 4,805
- base-only finding tuples: 0
- operator-only finding tuples: 19

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

## F-05: Scanner fixed versions are not drop-in remediations for shaded Hadoop dependencies

**Observation**

The exact Spark 4.0.4 base image contains `hadoop-client-runtime-3.4.1.jar`. Trivy attributed Jackson Core `2.12.7` and Jackson Databind `2.12.7.1` findings to this artifact.

Inspection of the Hadoop 3.4.1 source and Maven dependency graph established the dependency path:

`hadoop-client-runtime:3.4.1`
→ `hadoop-client:3.4.1`
→ `hadoop-common:3.4.1`
→ `jackson-databind:2.12.7.1`

The Hadoop runtime build relocates `com/` packages under `org.apache.hadoop.shaded.com/`. Inspection of the released artifact confirmed the embedded Maven metadata and relocated Jackson implementation, including:

`org/apache/hadoop/shaded/com/fasterxml/jackson/databind/ObjectMapper.class`

The frozen Trivy snapshot reported 13 Jackson vulnerability records against this Hadoop runtime:

- 3 against `jackson-core:2.12.7`
- 10 against `jackson-databind:2.12.7.1`

**Controlled remediation experiment**

A Databind-only override to a scanner-listed fixed version produced a mixed Jackson dependency family: Databind `2.18.11` with Core and Annotations `2.12.7`.

A coordinated Jackson `2.18.11` override aligned the pre-shading dependency graph, but packaging with Hadoop's existing Maven Shade Plugin `3.4.1` failed while processing a Java 21 multi-release class from Jackson Core:

`Unsupported class file major version 65`

Changing only the Shade Plugin version from `3.4.1` to `3.5.0` allowed the shaded runtime artifact to build successfully.

The first successfully packaged `2.18.11` candidate still contained a compatibility defect. Jackson `2.18.11` changed the JAXB dependency from:

`jakarta.xml.bind:jakarta.xml.bind-api`

to:

`javax.xml.bind:jaxb-api`

Hadoop 3.4.1 already excludes `javax.xml.bind:jaxb-api` from `hadoop-client-runtime`. As a result, the candidate contained the relocated Jackson JAXB module but zero relocated `javax.xml.bind` classes.

The packaged `JaxbAnnotationIntrospector` still referenced those classes. Runtime method resolution failed with `NoClassDefFoundError`, including under the exact Spark 4.0.4 Java 17 runtime.

Removing only that JAXB exclusion caused Hadoop dependency management to resolve `javax.xml.bind:jaxb-api:2.2.11`. The rebuilt artifact contained 113 relocated JAXB classes and the same targeted runtime-resolution probe passed.

This repaired prototype was then installed into a derived image based on the exact Spark 4.0.4 digest. In that image:

- Spark 4.0.4 started successfully on Java 17.0.19
- a local Spark job completed successfully
- Hadoop local filesystem initialization succeeded
- the targeted JAXB linkage probe succeeded

The whole-image scan using the same pinned Trivy version and frozen vulnerability databases changed from:

- OS findings: `4670 → 4670`
- Java findings: `173 → 160`

The exact normalized Java manifest diff contained 13 removed findings and zero added findings. All 13 removals were the Jackson findings attributed to `opt/spark/jars/hadoop-client-runtime-3.4.1.jar`.

**Security implication**

A scanner's `FixedVersion` field is not, by itself, a safe remediation instruction for a shaded dependency.

In this case, reaching a prototype with fewer scanner-reported findings and passing the targeted runtime checks required coordinated dependency changes, a build-tool upgrade, analysis of relocation rules, analysis of changed transitive dependency coordinates, adjustment of an existing exclusion, runtime linkage testing, Spark-level smoke testing, and a final whole-image rescan.

Replacing Spark's standalone Jackson JAR would also not replace this copy because the affected implementation is packaged inside the Hadoop runtime under a relocated namespace.

For inherited shaded dependencies, remediation must consider the parent artifact and its compatibility requirements rather than replacing individual JAR files directly.

**Limitations**

This experiment does not establish exploitability or runtime reachability of the original scanner findings.

The repaired artifact was produced through the `hadoop-client-runtime` module packaging path using released Hadoop 3.4.1 dependencies. It was not validated through a complete Hadoop source reactor build or the full Hadoop and Spark test suites.

The repaired runtime contained 113 relocated JAXB classes compared with 119 in the original artifact, so the two artifacts are not structurally identical.

The vulnerability results are specific to the recorded Trivy and database snapshot. Replacing a file in a later OCI layer also does not imply that the original bytes have been physically removed from inherited lower layers.

**Status**

Verified
