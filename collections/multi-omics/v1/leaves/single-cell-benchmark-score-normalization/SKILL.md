---
name: single-cell-benchmark-score-normalization
description: Use when you have a methods x metrics x datasets matrix of raw evaluation
  scores from simulating scRNA-seq/SRT data (8 metrics over 15 data properties) and
  must turn it into comparable per-method accuracy scores in [0,1].
when_to_use_negative:
- You are scoring functionality, scalability, or usability criteria rather than accuracy
  — this pipeline encodes the accuracy criterion computed from the 8 metric x data-property
  evaluations
- You have not yet run the simulation methods and lack the raw per-dataset metric
  scores; you need the task_006 output matrix first
- Your metric values are already on a comparable [0,1] scale with consistent orientation
  — a second standard-normal + min-max pass would distort rather than harmonize them
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: simpipe
  role: R package providing the standard pipeline and scoring utilities for evaluating
    simulated data and normalizing/aggregating the resulting metric scores
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Companion package collecting the 49 simulation methods whose raw metric scores
    feed the normalization; installed alongside simpipe via devtools
  repo: https://github.com/duohongrui/simmethods
- name: simpipe2docker
  role: Optional wrapper linking the R environment to the simpipe Docker container
    so normalization runs in a reproducible containerized environment
  repo: https://github.com/duohongrui/simpipe2docker
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:719f41b6b03a40d0bad3176ea61af5c91b689491eadea109155a89953b2dcf61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simpipe
  tools:
  - simmethods
  - simpipe
  - simpipe2docker
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# single-cell-benchmark-score-normalization

## Summary

Two-stage normalization and aggregation pipeline (implemented in the simpipe R package) that converts raw evaluation-metric scores from a single-cell/spatial transcriptomics simulation benchmark into one per-method accuracy score by standard-normal transformation, inversion of lower-is-better metrics, min-max rescaling, and hierarchical averaging.

## When to use

Use when you have a methods x metrics x datasets matrix of raw evaluation scores from simulating scRNA-seq/SRT data (8 metrics over 15 data properties) and must turn it into comparable per-method accuracy scores in [0,1].

## When NOT to use

- You are scoring functionality, scalability, or usability criteria rather than accuracy — this pipeline encodes the accuracy criterion computed from the 8 metric x data-property evaluations
- You have not yet run the simulation methods and lack the raw per-dataset metric scores; you need the task_006 output matrix first
- Your metric values are already on a comparable [0,1] scale with consistent orientation — a second standard-normal + min-max pass would distort rather than harmonize them

## Inputs

- Raw metric-score matrix from task_006: per-method, per-dataset scores of 8 evaluation metrics applied to 15 data properties (cell-level and gene-level) for the simulated vs. real scRNA-seq/SRT data

## Outputs

- normalized_scores.csv — all scores transformed to the (0,1) interval, inverted where lower-is-better, and min-max rescaled within each data property x metric combination
- accuracy_scores_aggregated.csv — one final accuracy score per simulation method (hierarchical mean over data properties, then datasets, then metrics)

## How to apply

Load the raw metric-score matrix produced by the simulation evaluation (methods x metrics x datasets) into R and process it with the simpipe scoring utilities. For each dataset, standardize each metric score to a standard-normal scale and push it through the normal CDF so values lie strictly within (0,1), avoiding boundary values of 0 or 1. For metrics where lower values indicate better performance (e.g. distance/error-type metrics), apply the 1-x inversion so that higher normalized values always mean better performance. Then apply min-max rescaling within each data property x metric combination to bring all normalized scores onto a common [0,1] scale. Finally aggregate hierarchically — average within each data property, then across datasets, then across metrics — to yield one final accuracy score per simulation method, writing normalized_scores.csv and accuracy_scores_aggregated.csv. This implements the accuracy criterion of the benchmark, which quantifies how well each of the 49 simulation methods reproduces real-data properties across the 152 reference datasets.

## Related tools

- **simpipe** (R package providing the standard pipeline and scoring utilities for evaluating simulated data and normalizing/aggregating the resulting metric scores) — https://github.com/duohongrui/simpipe
- **simmethods** (Companion package collecting the 49 simulation methods whose raw metric scores feed the normalization; installed alongside simpipe via devtools) — https://github.com/duohongrui/simmethods
- **simpipe2docker** (Optional wrapper linking the R environment to the simpipe Docker container so normalization runs in a reproducible containerized environment) — https://github.com/duohongrui/simpipe2docker

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- Every entry of normalized_scores.csv lies strictly within [0,1] (normal-CDF transform ensures no exact 0 or 1 before min-max rescaling)
- accuracy_scores_aggregated.csv contains exactly one accuracy score for each of the 49 evaluated simulation methods
- Lower-is-better metrics (distance/error-type) are inverted (1-x) so that higher normalized values always mean better performance
- Min-max rescaling is applied within each data property x metric combination, so scores are comparable across the 15 data properties
- Top-ranked methods after aggregation reproduce the paper's finding that SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy across platforms

## Limitations

- No method performs best on all criteria; the aggregated accuracy score alone can justify 'good-yet-not-the-best' choices, and users should consider trade-offs between accuracy and scalability (or functionality)
- Accuracy scoring reflects resemblance to real data only — methods like Phenopath, Lun, Simple, and MFA can score high on scalability while failing to generate realistic simulated data
- Metric coverage may miss properties some methods cannot capture (e.g. scDD cannot characterize genes with >=75% zeros; BASiCS does not characterize sparsity), so aggregated accuracy can mask property-specific failures

## Evidence

- [methods] devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe"): "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [readme] Simpipe provides the standard pipeline of estimating parameters from read data, simulating and evaluating simulated data.: "Simpipe provides the standard pipeline of estimating parameters from read data, simulating and evaluating simulated data."
- [results] The accuracy criterion is computed from 8 evaluation metrics applied to 15 data properties at the cell or gene level, quantifying the ability of a method to generate simulated data that resemble real data.: [evidence span withheld — anchor status: not_found]
- [abstract] We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability using 152 reference datasets derived from 24: "We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability using 152 reference datasets derived from 24"
