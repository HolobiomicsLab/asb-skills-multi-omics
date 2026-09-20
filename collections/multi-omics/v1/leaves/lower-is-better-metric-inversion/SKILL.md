---
name: lower-is-better-metric-inversion
description: Use when combining evaluation metrics for scRNA-seq/SRT simulation benchmarking
  where some metrics (e.g., distance/error metrics) improve as values decrease, and
  scores must be oriented so higher normalized values always mean better performance
  before aggregation.
when_to_use_negative:
- The metric is already higher-is-better — applying 1-x would invert a correctly oriented
  score and corrupt the aggregation
- Scores have not yet been standardized and transformed through the normal CDF — applying
  1-x to raw metric values on unbounded scales mixes incommensurable ranges
- The goal is assessing scalability or usability of methods rather than the accuracy
  criterion
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: simpipe
  role: Provides the R scoring utilities for standard-normal standardization, normal-CDF
    transformation, 1-x inversion of lower-is-better metrics, min-max rescaling, and
    hierarchical aggregation of accuracy scores
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Package that runs the 49 simulation methods whose metric scores feed the normalization/inversion
    pipeline
  repo: https://github.com/duohongrui/simmethods
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:74bf9110f63f4df801d5cf8b152cd68e690d12b9a9e5f988a126f7097267159c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simpipe
  tools:
  - simmethods
  - simpipe
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Lower-is-better metric inversion

## Summary

Inverts normalized metric scores where lower raw values indicate better performance (distance/error-type metrics) via 1-x, as part of the simpipe two-stage score normalization and aggregation that converts 8 evaluation metrics over 15 data properties into a single per-method accuracy score.

## When to use

Use when combining evaluation metrics for scRNA-seq/SRT simulation benchmarking where some metrics (e.g., distance/error metrics) improve as values decrease, and scores must be oriented so higher normalized values always mean better performance before aggregation.

## When NOT to use

- The metric is already higher-is-better — applying 1-x would invert a correctly oriented score and corrupt the aggregation
- Scores have not yet been standardized and transformed through the normal CDF — applying 1-x to raw metric values on unbounded scales mixes incommensurable ranges
- The goal is assessing scalability or usability of methods rather than the accuracy criterion

## Inputs

- Raw metric-score matrix from task_006 (simulation methods x evaluation metrics x 152 reference datasets)
- Mapping of which of the 8 evaluation metrics are lower-is-better (distance/error-type) vs higher-is-better
- Data property x metric combination structure over 15 data properties

## Outputs

- normalized_scores.csv with all entries within [0,1]
- accuracy_scores_aggregated.csv containing exactly one accuracy score per evaluated simulation method (49 methods)

## How to apply

Load the raw metric-score matrix produced by task_006 (methods x metrics x datasets) into R with the simpipe scoring utilities. For each dataset, standardize each metric score to a standard-normal scale and pass it through the normal CDF so normalized values lie strictly within (0,1), avoiding boundary values of 0 or 1. For metrics where lower values indicate better performance (e.g., distance/error-type metrics), apply the 1-x inversion so that higher normalized values always mean better performance; for higher-is-better metrics, skip the inversion. Then apply min-max rescaling within each data property x metric combination onto a common [0,1] scale, and aggregate hierarchically (average within data property, across datasets, then across metrics) to one accuracy score per simulation method, writing normalized_scores.csv and accuracy_scores_aggregated.csv.

## Related tools

- **simpipe** (Provides the R scoring utilities for standard-normal standardization, normal-CDF transformation, 1-x inversion of lower-is-better metrics, min-max rescaling, and hierarchical aggregation of accuracy scores) — https://github.com/duohongrui/simpipe
- **simmethods** (Package that runs the 49 simulation methods whose metric scores feed the normalization/inversion pipeline) — https://github.com/duohongrui/simmethods

## Examples

```
devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- Every entry in normalized_scores.csv lies within [0,1]
- accuracy_scores_aggregated.csv contains exactly one accuracy score for each of the evaluated simulation methods
- After inversion, higher normalized values correspond to better performance for distance/error-type metrics (low raw error maps to high normalized score)
- Min-max rescaling is applied within each data property x metric combination, so all combinations share a common [0,1] scale
- Boundary values of exactly 0 or 1 do not appear before inversion because normal-CDF transformation keeps values strictly within (0,1)

## Limitations

- Inversion assumes a monotone interpretation: applying 1-x only makes sense for metrics where lower values indicate better performance; misclassification of metric direction invalidates the accuracy score
- No simulation method performs best on all criteria, so the aggregated accuracy score must be interpreted alongside functionality, scalability, and usability trade-offs
- The pipeline inherits its input quality from task_006's raw metric-score matrix over 152 reference datasets from 24 platforms; errors there propagate through normalization

## Evidence

- [full_text] accuracy computed from 8 evaluation metrics applied to 15 data properties: [evidence span withheld — anchor status: not_found]
- [full_text] inversion and min-max normalization description: [evidence span withheld — anchor status: not_found]
- [full_text] validation statement from workflow: [evidence span withheld — anchor status: not_found]
- [readme] simpipe purpose from README: "Simpipe provides the standard pipeline of estimating parameters from read data, simulating and evaluating simulated data."
- [methods] installation of simpipe: "devtools::install_github("duohongrui/simpipe")"
