---
name: hierarchical-accuracy-score-aggregation
description: Use when raw metric x property scores from the 49-method benchmark exist
  and a single comparable accuracy score per simulation method (SRT and scRNA-seq
  simulators) must be produced.
when_to_use_negative:
- Scores are already min-max normalized within each property x metric block — re-applying
  the CDF-plus-rescaling stage would distort them.
- You need scalability, functionality, or usability scores; this pipeline only produces
  the accuracy criterion.
- The goal is comparing real reference datasets rather than scoring simulated-data
  fidelity for the 49 simulation methods.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: simpipe
  role: provides the scoring/normalization utilities used to standardize, invert,
    rescale, and hierarchically aggregate the metric-property scores
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: collection of 49 simulation methods whose per-method raw metric scores feed
    the aggregation pipeline
  repo: https://github.com/duohongrui/simmethods
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:7a8391a36dd7244d4794f889e9f920e9b98a08643087656fa59f880b485a6c0b
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# hierarchical-accuracy-score-aggregation

## Summary

Reconstructs the two-stage normalization and aggregation pipeline from the Genome Biology 2024 simulation benchmark: raw evaluation-metric scores over 15 data properties are transformed to strict (0,1) values, inverted for lower-is-better metrics, min-max rescaled, and hierarchically averaged into one accuracy score per simulation method.

## When to use

Use when raw metric x property scores from the 49-method benchmark exist and a single comparable accuracy score per simulation method (SRT and scRNA-seq simulators) must be produced.

## When NOT to use

- Scores are already min-max normalized within each property x metric block — re-applying the CDF-plus-rescaling stage would distort them.
- You need scalability, functionality, or usability scores; this pipeline only produces the accuracy criterion.
- The goal is comparing real reference datasets rather than scoring simulated-data fidelity for the 49 simulation methods.

## Inputs

- raw metric-score matrix (methods x metrics x datasets) from task_006, in R
- list of metric directions (higher-is-better vs. distance/error-type metrics requiring 1-x inversion)
- data property x metric combination definitions (15 data properties, 8 evaluation metrics)

## Outputs

- normalized_scores.csv (all entries strictly in [0,1])
- accuracy_scores_aggregated.csv (exactly one accuracy score per evaluated simulation method)

## How to apply

Load the raw metric-score matrix (methods x metrics x datasets) produced by task_006 into R and process it with the simpipe scoring utilities. For each dataset, standardize each metric score to a standard-normal scale and push it through the normal CDF so values fall strictly within (0,1), avoiding boundary 0/1. For lower-is-better metrics (distance/error type), apply 1-x inversion so higher always means better. Apply min-max rescaling within each data property x metric combination onto a common [0,1] scale, then aggregate hierarchically: average within each data property, then across datasets, then across metrics, writing normalized_scores.csv and accuracy_scores_aggregated.csv. This mirrors the accuracy criterion built from 8 evaluation metrics over 15 data properties used to compare the 49 simmethods simulators on 152 reference datasets.

## Related tools

- **simpipe** (provides the scoring/normalization utilities used to standardize, invert, rescale, and hierarchically aggregate the metric-property scores) — https://github.com/duohongrui/simpipe
- **simmethods** (collection of 49 simulation methods whose per-method raw metric scores feed the aggregation pipeline) — https://github.com/duohongrui/simmethods

## Examples

```
devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- Every entry of normalized_scores.csv lies within [0,1] with no exact 0 or 1 values (CDF transform guarantees strict (0,1) before rescaling).
- accuracy_scores_aggregated.csv contains exactly one accuracy score for each of the 49 evaluated simulation methods.
- Lower-is-better (distance/error) metrics show inverted orientation: after 1-x, higher normalized values always mean better performance.
- Ranking sanity check: top-accuracy methods should include SRTsim, scDesign3, ZINB-WaVE, and scDesign2 across platforms.
- Min-max rescaled scores for each data property x metric block span the full [0,1] range within a dataset.

## Limitations

- Accuracy alone is not sufficient for method selection: Phenopath, Lun, Simple, and MFA score high on scalability but cannot generate realistic simulated data, so accuracy results must be weighed against scalability/functionality trade-offs.
- No method performs best on all criteria; the aggregated accuracy score can mask criterion-specific weaknesses.
- Execution errors from failed parameter estimations or missing/infinite values in calculations can leave gaps in the raw metric matrix that the normalization step must handle.

## Evidence

- [abstract] The accuracy criterion is computed from 8 evaluation metrics applied to 15 data properties at the cell or gene level, quantifying the ability of a method to generate simulated data that resemble real data.: "We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability"
- [methods] simpipe supplies the standard pipeline utilities installed via devtools for the scoring workflow.: "devtools::install_github("duohongrui/simpipe")"
- [readme] simpipe's stated purpose covers evaluating simulated data, which is where the normalization/aggregation scoring utilities live.: "The goal of simpipe is to establish a standard pipeline to estimating
parameters from real datasets, simulating single-cell RNA-seq datasets
and evaluating the simulated data from general and"
- [abstract] Top accuracy performers that the aggregated scores should recover.: "SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across various platforms"
