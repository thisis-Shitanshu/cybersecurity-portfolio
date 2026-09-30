# Audit Plan

## Objective

Understand the Spark Operator controller image, its dependency surface, and its security responsibility boundaries before proposing documentation changes upstream.

## Questions

### Image architecture

- What base image does Spark Operator use?
- What files and packages does Kubeflow add?
- Why does the controller require Spark?
- Where is `spark-submit` invoked?

## Dependency ownership

- Which dependencies belong to Spark Operator?
- Which dependencies are inherited from Apache Spark?
- Which dependencies come from Hadoop or other ecosystem components?
- Which dependencies belong to the user's application image?

## Vulnerability remediation

- Can inherited vulnerable JARs be replaced independently?
- Are they transitively coupled?
- Are any of them shaded?
- What ABI/API compatibility risks exist?
- Would replacement require rebuilding the Spark base image?
- Can a replacement in a later OCI layer leave vulnerable content detectable in lower layers?

## Validation

- What tests would be required after changing a bundled dependency?
- Would Spark's full test suite need to run?
- Which Spark Operator integration tests would be relevant?
- Would independently maintaining such patches effectively create a downstream Spark distribution?
