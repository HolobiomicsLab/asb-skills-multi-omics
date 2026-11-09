---
name: ks-goodness-of-fit-testing
description: 'Use when judging simulator accuracy: you have a real reference expression
  matrix plus simulated matrices and need distribution-level distances between properties
  such as library size (LS), per-cell zero counts (FZC), or cell–cell correlation
  (CCC).'
when_to_use_negative:
- You are evaluating functionality, scalability, or usability of a simulator rather
  than distributional accuracy of its output (the paper scores these as separate criteria)
- You only have a single dataset (no real-vs-simulated pair) to compare, so no goodness-of-fit
  distance can be computed
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: simpipe
  role: Standard pipeline that computes the data-property vectors and the KS/MAD/MAE/RMSE/OV/BH
    and multiKS/KDE evaluation metrics between real and simulated data
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Package supplying the 49 simulation methods that generate the simulated expression
    matrices being evaluated
  repo: https://github.com/duohongrui/simmethods
- name: devtools
  role: Used to install simmethods and simpipe from GitHub before running the evaluation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:68012ba7e6d0288d8564441a6d82470fb1b098db1dae2fcf2cd07f9aed5bad7e
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ks-goodness-of-fit-testing

## Summary

A distribution-comparison step from the Simpipe accuracy module (Duo et al., Genome Biology 2024) that quantifies the discrepancy between per-cell or per-gene property distributions of real scRNA-seq/SRT reference data and simulated expression matrices using the Kolmogorov–Smirnov (KS) distance, alongside MAD, MAE, RMSE, OV, and BH metrics.

## When to use

Use when judging simulator accuracy: you have a real reference expression matrix plus simulated matrices and need distribution-level distances between properties such as library size (LS), per-cell zero counts (FZC), or cell–cell correlation (CCC).

## When NOT to use

- You are evaluating functionality, scalability, or usability of a simulator rather than distributional accuracy of its output (the paper scores these as separate criteria)
- You only have a single dataset (no real-vs-simulated pair) to compare, so no goodness-of-fit distance can be computed

## Inputs

- Real reference scRNA-seq/SRT expression count matrix (e.g., the Simpipe vignette example reference)
- One or more simulated expression matrices produced by Simmethods simulators
- R environment with devtools, simpipe, and simmethods installed from GitHub

## Outputs

- metric_scores.csv with one row per data property (15) × metric (6 one-D: MAD, KS, MAE, RMSE, OV, BH; 2 two-D: multiKS, KDE) for each simulated dataset
- Per-method accuracy profile showing which of the 15 data properties each simulator reproduces well

## How to apply

Install simmethods and simpipe from GitHub with devtools::install_github(), then load the real reference expression matrix shipped with the Simpipe vignette and the Simmethods-simulated counterparts. For each of the 15 data properties (LS, FZC, CCC, TMM, ELS, FCO, RLZ, ME, SD, CV, FZG, FGO, RMS, RMZ, RDM), compute the property vector on the real matrix and on every simulated matrix. For 1-D per-cell/per-gene properties, compute the KS distance together with MAD, MAE, RMSE, OV, and BH between the real and simulated distributions; for 2-D pairwise/correlation properties, use the multiKS statistic and the KDE-based distance instead. Rationale: the paper scores each simulator per data property, showing that different methods capture different properties well, so the KS-based distance must be reported per property × metric, not as a single aggregate. Assemble all values into a metric_scores.csv table (one row per data-property × metric × simulated dataset) and verify all metric values are finite, since the paper identifies missing or infinite values as the main source of execution errors.

## Related tools

- **simpipe** (Standard pipeline that computes the data-property vectors and the KS/MAD/MAE/RMSE/OV/BH and multiKS/KDE evaluation metrics between real and simulated data) — https://github.com/duohongrui/simpipe
- **simmethods** (Package supplying the 49 simulation methods that generate the simulated expression matrices being evaluated) — https://github.com/duohongrui/simmethods
- **devtools** (Used to install simmethods and simpipe from GitHub before running the evaluation)

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- metric_scores.csv exists and contains rows for all 15 data properties crossed with the six 1-D metrics and the two 2-D metrics for every simulated dataset
- All metric values in metric_scores.csv are finite — no NA, NaN, or Inf (the paper names missing/infinite values as the main source of execution errors)
- KS distances are reported per data property and per simulated dataset, not aggregated into a single score, preserving the per-property comparison
- Metrics computed for a simulated matrix that closely matches the reference are near zero; large KS values flag properties (e.g., library size, sparsity) that the simulator fails to capture
- Each metric value is traceable to a property computed from both the real reference matrix and the corresponding simulated matrix

## Limitations

- Execution errors are mainly caused by failed parameter estimations and the appearance of missing or infinite values in the calculations
- Different simulation methods capture different data properties well, so a low KS distance on one property does not imply overall accuracy
- No method performs best on all criteria, so the KS-based accuracy scores must be weighed against scalability and functionality trade-offs

## Evidence

- [abstract] execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations: "execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations"
- [methods] devtools::install_github("duohongrui/simmethods") and devtools::install_github("duohongrui/simpipe"): "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [abstract] a standard pipeline Simpipe ( https://github.com/duohongrui/simpipe ; https://doi.org/10.5281/zenodo.11178409 ): "a standard pipeline Simpipe ( https://github.com/duohongrui/simpipe ; https://doi.org/10.5281/zenodo.11178409 )"
- [readme] The goal of simpipe is to establish a standard pipeline to estimating parameters from real datasets, simulating single-cell RNA-seq datasets and evaluating the simulated data from general and functional perspectives.: "The goal of simpipe is to establish a standard pipeline to estimating
parameters from real datasets, simulating single-cell RNA-seq datasets
and evaluating the simulated data from general and"
