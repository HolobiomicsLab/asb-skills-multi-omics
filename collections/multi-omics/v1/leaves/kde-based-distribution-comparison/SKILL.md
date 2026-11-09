---
name: kde-based-distribution-comparison
description: 'Use when scoring the accuracy of scRNA-seq or SRT simulation methods:
  the data property under evaluation is 2-D (e.g., cell–cell correlation, CCC) and
  you need the KDE-based distance between real and simulated distributions.'
when_to_use_negative:
- The property is 1-D (per-cell or per-gene, e.g., library size, fraction of zeros
  per gene) — use MAD, KS, MAE, RMSE, OV, or BH instead of the KDE-based distance
- The goal is functionality, scalability, or usability scoring rather than accuracy
  evaluation of simulated data
- Simulated matrices contain missing or infinite values that have not been resolved,
  since the paper reports execution errors are mainly caused by missing/infinite values
  in calculations
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: simpipe
  role: Standard pipeline that computes the 15 data properties and the accuracy metrics,
    including the KDE-based distance, between real and simulated data
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Provides the simulated expression matrices (49 collected simulation methods)
    that are compared against real references
  repo: https://github.com/duohongrui/simmethods
- name: devtools
  role: Used to install simmethods and simpipe from GitHub
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:b18e785f6cc11b9c5efc6daa89799b150e595c0973e7111811f83c8feb15f629
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simpipe
  tools:
  - devtools
  - simmethods
  - simpipe
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# KDE-based distribution comparison for 2-D data properties

## Summary

Computes a kernel-density-estimation (KDE)-based distance between real and simulated single-cell/spatial transcriptomics expression matrices for two-dimensional (pairwise/correlation-type) data properties, one of the eight accuracy metrics used in the Simpipe benchmark to judge whether simulated data resemble real data.

## When to use

Use when scoring the accuracy of scRNA-seq or SRT simulation methods: the data property under evaluation is 2-D (e.g., cell–cell correlation, CCC) and you need the KDE-based distance between real and simulated distributions.

## When NOT to use

- The property is 1-D (per-cell or per-gene, e.g., library size, fraction of zeros per gene) — use MAD, KS, MAE, RMSE, OV, or BH instead of the KDE-based distance
- The goal is functionality, scalability, or usability scoring rather than accuracy evaluation of simulated data
- Simulated matrices contain missing or infinite values that have not been resolved, since the paper reports execution errors are mainly caused by missing/infinite values in calculations

## Inputs

- Real reference expression matrix (e.g., the Simpipe vignette example reference dataset)
- Simulated expression matrices produced by Simmethods methods
- 2-D (pairwise/correlation) property matrices computed from real and simulated data, e.g., CCC

## Outputs

- KDE-based distance value per 2-D data property per simulated dataset
- Rows in metric_scores.csv for all 15 data properties crossed with the two 2-D metrics (multiKS and KDE-based distance) per simulated dataset

## How to apply

Install the simmethods and simpipe R packages with devtools::install_github, then load the Simpipe vignette real reference expression matrix and the Simmethods-simulated counterpart matrices. Compute each 2-D data property (e.g., CCC, one of the 15 properties LS, FZC, CCC, TMM, ELS, FCO, RLZ, ME, SD, CV, FZG, FGO, RMS, RMZ, RDM) as a property matrix on the real reference and on every simulated matrix. For 2-D properties, apply the multiKS statistic and the KDE-based distance between real and simulated data, in contrast to 1-D properties which use MAD, KS, MAE, RMSE, OV, and BH. Assemble one row per data-property x metric for each simulated dataset into metric_scores.csv, ensuring all values are finite — the paper identifies the appearance of missing or infinite values in calculations as the main source of execution errors.

## Related tools

- **simpipe** (Standard pipeline that computes the 15 data properties and the accuracy metrics, including the KDE-based distance, between real and simulated data) — https://github.com/duohongrui/simpipe
- **simmethods** (Provides the simulated expression matrices (49 collected simulation methods) that are compared against real references) — https://github.com/duohongrui/simmethods
- **devtools** (Used to install simmethods and simpipe from GitHub)

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- metric_scores.csv exists and contains rows for all 15 data properties crossed with the six 1-D metrics and the two 2-D metrics (multiKS and KDE-based distance) for every simulated dataset
- All metric values are finite — no missing or infinite values, which the paper identifies as the main source of execution errors
- KDE-based distance is applied only to the 2-D properties (e.g., CCC), while 1-D properties use MAD, KS, MAE, RMSE, OV, and BH
- Each simulated dataset yields one row per data-property x metric, so no property/metric combinations are dropped

## Limitations

- Execution errors are mainly caused by failed parameter estimations and the appearance of missing or infinite values in calculations
- Different methods capture different data properties well, so a single 2-D metric distance does not determine overall method quality — no method performs best on all criteria
- The metrics depend on the simpipe/simmethods R ecosystem (GitHub-installed, GPL-3.0), with a Docker route via simpipe2docker for reproducible environments

## Evidence

- [abstract] execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations: "execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations"
- [methods] install simpipe via devtools from GitHub: "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [readme] simpipe purpose statement: "establish a standard pipeline to estimating parameters from real datasets, simulating single-cell RNA-seq datasets"
- [other] task card validation criterion for metric_scores.csv: [evidence span withheld — anchor status: not_found]
