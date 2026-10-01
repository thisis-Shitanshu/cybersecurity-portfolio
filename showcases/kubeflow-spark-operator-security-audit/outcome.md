# Outcome

## Upstream contribution

This investigation supports Kubeflow Spark Operator issue
[#3143](https://github.com/kubeflow/spark-operator/issues/3143).

A documentation change was prepared to explain:

- the security boundary between Spark Operator and its Spark base image
- how inherited dependencies affect vulnerability reports
- why scanner-reported fixed versions require compatibility validation
- when rebuilding a parent Spark or Hadoop artifact may be necessary
- the existing custom-image workflow for users with stricter security requirements

## Engineering result

The investigation traced vulnerability findings from the final controller image back to their source and tested one difficult remediation path involving a shaded Hadoop dependency.

The experiment showed that updating an embedded dependency can affect:

- dependency alignment
- Maven build tooling
- transitive dependencies
- shaded package contents
- runtime class resolution
- Spark compatibility

The final prototype passed targeted runtime checks and reduced the Jackson findings attributed to the Hadoop runtime in the recorded vulnerability scan.

It was not treated as a production-ready Hadoop patch because the complete Hadoop and Spark test suites were outside the scope of the experiment.

## Upstream pull request

[#XXXX](https://github.com/kubeflow/spark-operator/pull/XXXX)
