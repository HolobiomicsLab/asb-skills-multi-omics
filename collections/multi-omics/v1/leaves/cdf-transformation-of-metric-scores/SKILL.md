---
name: cdf-transformation-of-metric-scores
description: Use when you have a raw matrix of metric-by-dataset scores from evaluating
  simulated single-cell/spatial data against real reference data and need comparable,
  bounded per-method accuracy scores.
when_to_use_negative:
- Inputs are raw simulated count matrices rather than already-computed metric scores
  — parameter estimation and simulation should be handled first by simmethods/simpipe
  simulation steps
- The task is ranking methods by scalability, functionality, or usability rather than
  accuracy, since those criteria have separate scoring pipelines in the benchmark
- You need best-on-all-criteria method selection — the paper notes no method performs
  best on all criteria and recommends good-yet-not-the-best methods chosen for the
  specific scenario
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: simpipe
  role: R package providing the standard pipeline and scoring utilities for normalizing
    and aggregating evaluation-metric scores into accuracy scores
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Companion collection of the 49 simulation methods whose metric scores are
    the raw inputs to the CDF transformation and aggregation
  repo: https://github.com/duohongrui/simmethods
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:0f9aa4d6ee66c65cee122f1f26ca842c012aa1208ccd12aa5fa40fcb4660d69f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simpipe
  tools:
  - simmethods
  - simpipe
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# CDF transformation of metric scores

## Summary

A two-stage score normalization and aggregation step from the simpipe benchmark pipeline that converts raw evaluation-metric scores (8 metrics over 15 data properties from 152 reference datasets) into per-method accuracy scores for 49 single-cell and spatial transcriptomics simulation methods.

## When to use

Use when you have a raw matrix of metric-by-dataset scores from evaluating simulated single-cell/spatial data against real reference data and need comparable, bounded per-method accuracy scores.

## When NOT to use

- Inputs are raw simulated count matrices rather than already-computed metric scores — parameter estimation and simulation should be handled first by simmethods/simpipe simulation steps
- The task is ranking methods by scalability, functionality, or usability rather than accuracy, since those criteria have separate scoring pipelines in the benchmark
- You need best-on-all-criteria method selection — the paper notes no method performs best on all criteria and recommends good-yet-not-the-best methods chosen for the specific scenario

## Inputs

- Raw metric-score matrix (simulation methods x evaluation metrics x datasets) from task_006, covering 8 evaluation metrics applied to 15 data properties at cell or gene level
- R environment with simpipe installed from GitHub

## Outputs

- normalized_scores.csv (all entries within [0,1])
- accuracy_scores_aggregated.csv (exactly one accuracy score per evaluated simulation method)

## How to apply

Load the raw metric-score matrix produced by task_006 (methods x metrics x datasets) into R and process it with the simpipe scoring utilities. For each dataset, standardize each metric score to a standard-normal scale and pass it through the normal CDF so values fall strictly within (0,1), avoiding boundary values of 0 or 1. For metrics where lower is better (distance/error-type metrics), apply the 1-x inversion so that higher normalized values always mean better performance. Then apply min-max rescaling within each data property x metric combination onto a common [0,1] scale, and aggregate hierarchically — average within each data property, then across datasets, then across metrics — to yield one final accuracy score per simulation method, writing normalized_scores.csv and accuracy_scores_aggregated.csv.

## Related tools

- **simpipe** (R package providing the standard pipeline and scoring utilities for normalizing and aggregating evaluation-metric scores into accuracy scores) — https://github.com/duohongrui/simpipe
- **simmethods** (Companion collection of the 49 simulation methods whose metric scores are the raw inputs to the CDF transformation and aggregation) — https://github.com/duohongrui/simmethods

## Examples

```
devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- Every entry in normalized_scores.csv lies within [0,1] after CDF transformation, inversion, and min-max rescaling
- accuracy_scores_aggregated.csv contains exactly one accuracy score for each of the evaluated simulation methods
- For distance/error-type metrics, higher normalized values correspond to better (lower raw) performance after 1-x inversion
- No normalized value equals exactly 0 or 1 before final min-max rescaling, confirming the normal-CDF (0,1) mapping worked
- The hierarchical averaging order (data property -> dataset -> metric) is respected so one score per method per aggregation level results

## Limitations

- No single accuracy score captures all criteria — the paper recommends weighing accuracy against scalability (or functionality) rather than optimizing accuracy alone
- Methods with high scalability scores (Phenopath, Lun, Simple, MFA) may still fail the accuracy assessment because they cannot generate realistic simulated data
- Execution errors from failed parameter estimation or missing/infinite values in calculations can leave a method without complete metric scores, limiting the normalization input

## Evidence

- [results] Standardize metric scores via normal CDF: [evidence span withheld — anchor status: not_found]
- [abstract] 49 methods, four criteria evaluated: "We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability using 152 reference datasets derived from 24"
- [readme] simpipe installation: "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [abstract] No method best on all criteria: "No method performs best on all criteria, thus a good-yet-not-the-best method is recommended if it solves problems effectively and reasonably"
