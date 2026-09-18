---
name: single-cell-expression-data-property-quantification
description: Use when you have a real reference scRNA-seq/SRT expression matrix plus
  simulated counterpart matrices and need to score which data properties the simulator
  reproduces, as in the Simpipe accuracy module.
when_to_use_negative:
- You only need to simulate data from a real reference without evaluating fidelity
  — parameter estimation/simulation with simmethods alone suffices
- Your comparison targets are functional/cell-type-preservation aspects of simulated
  data rather than the 15 numerical data properties scored here
- The real and simulated matrices are not on comparable scales or gene/cell dimensions,
  so property vectors are not comparable
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: simpipe
  role: Standard R pipeline used to compute the 15 data properties on real and simulated
    matrices and evaluate them with the accuracy metrics
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: R package that generates the simulated counterpart expression matrices evaluated
    against the real reference
  repo: https://github.com/duohongrui/simmethods
- name: devtools
  role: Used to install simmethods and simpipe from GitHub before running the property-quantification
    workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:96b5b82a122e9f4273385621f7a58abf61039dfddda06f33be4a83a95c7e69a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simpipe
  tools:
  - devtools
  - simmethods
  - simpipe
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-expression-data-property-quantification

## Summary

Quantify accuracy-relevant properties of single-cell RNA-seq or spatial transcriptomics expression matrices by computing 15 data properties (e.g., CCC, FCO, LS) on real reference and simulated data, then scoring their agreement with 1-D metrics (MAD, KS, MAE, RMSE, OV, BH) and 2-D metrics (multiKS, KDE-based distance) via the Simpipe R package.

## When to use

Use when you have a real reference scRNA-seq/SRT expression matrix plus simulated counterpart matrices and need to score which data properties the simulator reproduces, as in the Simpipe accuracy module.

## When NOT to use

- You only need to simulate data from a real reference without evaluating fidelity — parameter estimation/simulation with simmethods alone suffices
- Your comparison targets are functional/cell-type-preservation aspects of simulated data rather than the 15 numerical data properties scored here
- The real and simulated matrices are not on comparable scales or gene/cell dimensions, so property vectors are not comparable

## Inputs

- Real reference single-cell RNA-seq or spatially resolved transcriptomics expression matrix (as in the Simpipe vignette example)
- Simulated expression matrices produced by Simmethods simulation methods
- List of 15 data properties (LS, FZC, CCC, TMM, ELS, FCO, RLZ, ME, SD, CV, FZG, FGO, RMS, RMZ, RDM)

## Outputs

- metric_scores.csv with one row per data-property x metric for each simulated dataset
- Metric values for the six 1-D metrics (MAD, KS, MAE, RMSE, OV, BH) and the two 2-D metrics (multiKS, KDE-based distance)

## How to apply

Install the simmethods and simpipe R packages from GitHub with devtools, and load the Simpipe vignette real reference expression matrix and its simulated counterparts shipped with the packages. For each of the 15 data properties (LS, FZC, CCC, TMM, ELS, FCO, RLZ, ME, SD, CV, FZG, FGO, RMS, RMZ, RDM), compute the property vector on the real reference matrix and on every simulated matrix. For 1-D (per-cell or per-gene) property distributions, compute the MAD, KS, MAE, RMSE, OV, and BH distances between the real and simulated distributions; for 2-D (pairwise/correlation) properties, compute the multiKS statistic and a KDE-based distance. Assemble one row per data-property x metric for each simulated dataset and write metric_scores.csv. All metric values must be finite, because the paper identifies missing or infinite values in calculations as the main source of execution errors.

## Related tools

- **simpipe** (Standard R pipeline used to compute the 15 data properties on real and simulated matrices and evaluate them with the accuracy metrics) — https://github.com/duohongrui/simpipe
- **simmethods** (R package that generates the simulated counterpart expression matrices evaluated against the real reference) — https://github.com/duohongrui/simmethods
- **devtools** (Used to install simmethods and simpipe from GitHub before running the property-quantification workflow)

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- metric_scores.csv exists and contains rows for all 15 data properties crossed with the six 1-D metrics (MAD, KS, MAE, RMSE, OV, BH) for every simulated dataset
- 2-D properties (e.g., CCC) additionally have rows for the two 2-D metrics (multiKS and KDE-based distance)
- All metric values are finite — no missing (NA) or infinite (Inf) values, which the paper names as the main source of execution errors
- Each data property is computed consistently on both the real reference matrix and each simulated matrix before distance scoring
- Lower metric values for a property indicate the simulation method captured that data property better (different methods capture different properties well)

## Limitations

- Execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations
- Different methods capture different data properties well, so no single property or metric should be used to rank simulators overall
- Users should consider trade-offs between method accuracy and scalability (or functionality) when interpreting the scores

## Evidence

- [readme] Simpipe provides the standard pipeline of estimating parameters from read data, simulating single-cell RNA-seq datasets and evaluating the simulated data from general and functional perspectives.: [evidence span withheld — anchor status: not_found]
- [readme] devtools::install_github("duohongrui/simmethods"): "devtools::install_github("duohongrui/simmethods")"
- [abstract] execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations: "execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations"
- [abstract] a standard pipeline Simpipe ( https://github.com/duohongrui/simpipe ; https://doi.org/10.5281/zenodo.11178409 ): "a standard pipeline Simpipe ( https://github.com/duohongrui/simpipe ; https://doi.org/10.5281/zenodo.11178409 )"
