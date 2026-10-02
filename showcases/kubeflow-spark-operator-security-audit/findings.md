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

## F-06: Controller base-image selection can substantially reduce scanner-visible vulnerability surface

**Observation**

The Spark Operator controller image inherits most of its runtime surface from the selected Spark base image. This creates an opportunity to reduce scanner-visible findings through base-image selection without changing the Spark Operator binary itself.

A controlled comparison was performed across the current Spark 4.0.4 standard image and thinner Scala-only Spark variants.

All controller images were built from the same Spark Operator source revision:

`927676cd94197a772374c73351398df0a79625db`

The resulting `/usr/bin/spark-operator` binary was identical across all tested controller images:

`7c1f2c2cb988a4c4a3fd23e057852ce256307f0a9d2f8f1a4ee95a14d64fd29d`

### Compared controller images

| Controller base                        | Final image size | OS findings | Java findings | Go findings | Raw findings | Distinct vulnerability IDs |
| -------------------------------------- | ---------------: | ----------: | ------------: | ----------: | -----------: | -------------------------: |
| Apache Spark 4.0.4 standard            |    798,548,576 B |       4,670 |           173 |          19 |        4,862 |                      4,532 |
| Apache Spark 4.0.4 Scala-only          |    684,546,653 B |         258 |           173 |          19 |          450 |                        238 |
| Apache Spark 4.2.0 Scala-only          |    712,013,475 B |         285 |           118 |          19 |          422 |                        203 |
| Docker Official Spark 4.2.0 Scala-only |    711,688,209 B |          85 |           118 |          19 |          222 |                        112 |

> The scans used the same frozen Trivy 0.74.0 vulnerability databases used elsewhere in this audit. Counts represent raw scanner findings/occurrences, not independently verified exploitable vulnerabilities.

**Controlled 4.0.4 comparison**

The Apache Spark 4.0.4 standard and Scala-only images retained the same major Spark runtime stack:

- Spark 4.0.4
- Scala 2.13.16
- Java 17.0.19
- Hadoop client API/runtime 3.4.1
- 276 Spark JARs
- `/opt/spark` as `SPARK_HOME`
- UID/GID 185 for the `spark` user

The Scala-only image did not include Python.

Moving from the standard to Scala-only base reduced final-controller raw findings from 4,862 to 450, approximately 90.7%, while Java findings remained unchanged at 173 and Go binary findings remained unchanged at 19.

This indicates that the reduction was primarily associated with the smaller OS package surface rather than changes to the inherited Spark/Hadoop Java dependency set.

**Spark 4.2 comparison**

The Apache and Docker Official Spark 4.2 Scala-only images both provided:

- Spark 4.2.0
- Spark revision `32f7299601108917fb01920a54e084595b7b3bf8`
- Scala 2.13.18
- Hadoop client API/runtime 3.5.0
- 276 Spark JARs
- no Python installation

The Apache image used Java 21.0.11, while the Docker Official image used Java 21.0.12.1.

The Docker Official Spark 4.2 Scala-only controller produced 222 raw findings:

- 85 OS findings
- 118 Java findings
- 19 Go binary findings
- 112 distinct vulnerability IDs

This was the smallest scanner-visible result among the controller images tested.

**Functional validation**

The tested Scala-only controller images successfully preserved the runtime contract required by the Spark Operator Dockerfile and controller submission path, including:

- `spark-submit`
- JVM runtime
- Bash
- `apt-get`
- `libnss_wrapper`
- Spark runtime JARs
- execution as UID/GID 185
- installation and execution of `catatonit`

Local SparkPi execution completed successfully for the tested final controller images.

The Apache Spark 4.0.4 Scala-only controller also successfully submitted and completed a SparkApplication on Kubernetes.

For the Spark 4.2 candidates, the workload image was held constant at the exact Apache Spark 4.2.0 digest:

`docker.io/apache/spark:4.2.0@sha256:fc64959c04bd87b0ac686be9aaa9008b69cdb1afc695400528279c8b01f43d89`

This allowed the controller base image to remain the primary variable during the Kubernetes comparison.

The Docker Official Spark 4.2 Scala-only controller successfully:

- started the Spark Operator controller
- started the Spark Operator webhook
- generated and stored webhook TLS material
- updated mutating and validating webhook CA bundles
- served the admission webhook as UID/GID 185
- operated with admission `failurePolicy: Fail`
- admitted the SparkApplication
- submitted the Spark workload
- created the driver and executor
- completed SparkPi with driver exit code 0
- reached SparkApplication state `COMPLETED`

The webhook-enabled validation completed without observed webhook errors.

**Spark 4.2 compatibility observation**

Spark 4.2 introduced an additional Kubernetes NetworkPolicy feature step in the submission path.

With the current test deployment's controller RBAC, an unmodified Spark 4.2 submission failed because the controller service account could not patch:

`networkpolicies.networking.k8s.io`

The same Spark 4.2 controller and workload completed successfully after excluding only:

`org.apache.spark.deploy.k8s.features.NetworkPolicyFeatureStep`

This separates the observed failure from the base-image experiment: the tested Spark 4.2 controller runtime was functional, while the default Spark 4.2 Kubernetes behavior exposed an additional RBAC requirement in the current chart configuration.

This compatibility behavior should be evaluated separately from the security image-selection finding.

**Security interpretation**

The experiment demonstrates that image selection can substantially change scanner-visible controller exposure without modifying the Spark Operator binary.

It does not establish that:

- every reported vulnerability is exploitable in the Spark Operator context
- fewer scanner findings directly imply proportionally lower security risk
- Scala-only images are drop-in replacements for every Spark Operator workflow
- Python-dependent integration or end-to-end tests remain compatible
- Spark 4.2 is fully compatible with the current chart without additional configuration or RBAC changes
- the lowest-count image should automatically become the upstream default

The result instead supports treating the Spark base image as an explicit security and compatibility boundary. A smaller runtime image may reduce inherited package exposure, but image selection must be evaluated together with Spark version, Java version, Hadoop dependencies, supported workloads, admission behavior, and Kubernetes permissions.

**Status:**

Confirmed through controlled image comparison and functional validation
