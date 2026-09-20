---
name: min-max-scaling-per-data-property
description: Use when raw metric scores from simulated vs. real scRNA-seq/SRT comparisons
  sit on incommensurate scales (e.g., distance/error metrics vs. correlation-like
  metrics) and must be brought onto a common [0,1] scale before aggregating into a
  per-method accuracy score.
when_to_use_negative:
- Scores already share a common [0,1] scale and only hierarchical aggregation is needed
  — re-scaling would be redundant
- The criterion being scored is scalability or usability rather than accuracy — this
  skill normalizes the 8 accuracy evaluation metrics over 15 data properties
- Metric scores are missing/infinite because of failed parameter estimation — errors
  must be resolved before scaling, since infinite values break normalization
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: simpipe
  role: R pipeline whose scoring utilities perform the standardization, CDF transform,
    1-x inversion, min-max rescaling, and hierarchical aggregation of metric-property
    scores
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: R package that runs the 49 simulation methods whose evaluation metric scores
    feed the normalization pipeline
  repo: https://github.com/duohongrui/simmethods
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:6a422edd424a9b5935a6e714a8a7efe8e3161ab5b9217fd23f4b7ddb812d1998
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simpipe
  tools:
  - simmethods
  - simpipe
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# min-max-scaling-per-data-property

## Summary

A score-normalization step from the simpipe benchmark pipeline (Genome Biology 2024) that min-max rescales metric-property scores within each data property x metric combination so the 8 evaluation metrics over 15 data properties can be hierarchically aggregated into one accuracy score per simulation method.

## When to use

Use when raw metric scores from simulated vs. real scRNA-seq/SRT comparisons sit on incommensurate scales (e.g., distance/error metrics vs. correlation-like metrics) and must be brought onto a common [0,1] scale before aggregating into a per-method accuracy score.

## When NOT to use

- Scores already share a common [0,1] scale and only hierarchical aggregation is needed — re-scaling would be redundant
- The criterion being scored is scalability or usability rather than accuracy — this skill normalizes the 8 accuracy evaluation metrics over 15 data properties
- Metric scores are missing/infinite because of failed parameter estimation — errors must be resolved before scaling, since infinite values break normalization

## Inputs

- Raw metric-score matrix produced by task_006 (simulation methods x evaluation metrics x reference datasets)
- Mapping of which of the 15 data properties each metric-property score belongs to
- Flag indicating per metric whether higher-is-better or lower-is-better (distance/error-type metrics)

## Outputs

- normalized_scores.csv (all entries within [0,1], strictly inside (0,1) before min-max rescaling)
- accuracy_scores_aggregated.csv (one final accuracy score per evaluated simulation method)

## How to apply

Load the raw metric-score matrix (methods x metrics x datasets, e.g., from task_006) into R and process it with the simpipe scoring utilities. For each dataset, first standardize each metric score to a standard-normal scale and push it through the normal CDF so values lie strictly within (0,1), avoiding boundary values of 0 or 1. For metrics where lower values indicate better performance (distance/error-type metrics), apply the 1-x inversion so that higher normalized values always mean better performance. Then apply min-max rescaling within each data property x metric combination, which is this skill's core operation, to place all normalized scores on a common [0,1] scale. Finally support hierarchical aggregation: average within each data property, then across datasets, then across metrics, writing normalized_scores.csv and accuracy_scores_aggregated.csv.

## Related tools

- **simpipe** (R pipeline whose scoring utilities perform the standardization, CDF transform, 1-x inversion, min-max rescaling, and hierarchical aggregation of metric-property scores) — https://github.com/duohongrui/simpipe
- **simmethods** (R package that runs the 49 simulation methods whose evaluation metric scores feed the normalization pipeline) — https://github.com/duohongrui/simmethods

## Examples

```
devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- Every entry of normalized_scores.csv lies within [0,1]
- accuracy_scores_aggregated.csv contains exactly one accuracy score for each of the evaluated simulation methods (49 methods in the published benchmark)
- After the normal-CDF transform, no score equals exactly 0 or 1 (boundary values are avoided)
- Distance/error-type metrics have been 1-x inverted so that higher normalized values always mean better performance
- Aggregation order holds: property-level averages first, then across the 152 reference datasets, then across the 8 metrics

## Limitations

- Execution errors in the upstream evaluation can arise from failed parameter estimations and from missing or infinite values in calculations, which would corrupt the normalization inputs
- The min-max scale is defined within each data property x metric combination, so cross-combination comparability relies on the subsequent hierarchical averaging rather than on the scaling itself

## Evidence

- [results] The accuracy criterion is computed from 8 evaluation metrics applied to 15 data properties at the cell or gene level, quantifying the ability of a method to generate simulated data that resemble real data: [evidence span withheld — anchor status: not_found]
- [methods] install: "devtools::install_github("duohongrui/simpipe")"
- [readme] readme: "establish a standard pipeline to estimating parameters from real datasets, simulating single-cell RNA-seq datasets and evaluating the simulated data"
- [abstract] abstract scope: "We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability using 152 reference datasets"
