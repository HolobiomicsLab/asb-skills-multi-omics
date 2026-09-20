---
name: scalability-benchmark-visualization
description: Use when selecting among simulation methods for scRNA-seq or spatially
  resolved transcriptomics data and you need to quantify how execution time and memory
  scale with the number of cells or genes before committing to a method.
when_to_use_negative:
- When the goal is accuracy evaluation rather than computational scalability — fast
  methods such as Phenopath, Lun, Simple, and MFA score high on scalability but cannot
  generate realistic simulated data, so scalability alone must not drive method choice.
- When the input is already simulated data or a simulated count matrix; this skill
  benchmarks simulator execution, not downstream analysis of simulated outputs.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: simmethods
  role: Provides the 49 collected simulation methods (including dyntoy, ZINB-WaVE)
    and parameter estimation from real data that are benchmarked for scalability
  repo: https://github.com/duohongrui/simmethods
- name: simpipe
  role: Standard pipeline used to run each simulation method on resampled datasets
    and record execution time and peak memory
  repo: https://github.com/duohongrui/simpipe
- name: SCAM
  role: Shape-constrained additive model regressor used to predict execution time
    and peak memory and compute predicted-versus-actual Pearson correlations
- name: dyntoy
  role: Reference case with known quadratic time complexity in parameter estimation
    and quadratic memory complexity in data simulation, used to validate the scaling
    analysis
  repo: https://github.com/dynverse/dyntoy
- name: simpipe2docker
  role: Links the R environment to the simpipe Docker container so resource measurements
    run in a standardized environment
  repo: https://github.com/duohongrui/simpipe2docker
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:897da2b3ba8b398a399f7477ebfd245af99fce39eecf8aaf6e80a25103d5e301
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simmethods
  tools:
  - SCAM
  - SRTsim
  - ZINB-WaVE
  - dyntoy
  - scDesign2
  - scDesign3
  - simmethods
  - simpipe
  - simpipe2docker
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# scalability-benchmark-visualization

## Summary

Benchmarks the computational scalability of scRNA-seq/SRT data simulation methods by resampling reference datasets across a grid of sizes (100-10,000 cells x 500-10,000 genes), recording wall-clock execution time and peak memory per run, and fitting complexity models (y ~ log(x)+sqrt(x)+x+x^2+x^3) to assign each method a complexity class.

## When to use

Use when selecting among simulation methods for scRNA-seq or spatially resolved transcriptomics data and you need to quantify how execution time and memory scale with the number of cells or genes before committing to a method.

## When NOT to use

- When the goal is accuracy evaluation rather than computational scalability — fast methods such as Phenopath, Lun, Simple, and MFA score high on scalability but cannot generate realistic simulated data, so scalability alone must not drive method choice.
- When the input is already simulated data or a simulated count matrix; this skill benchmarks simulator execution, not downstream analysis of simulated outputs.

## Inputs

- Reference scRNA-seq/SRT datasets loadable via simmethods/simpipe (seven reference datasets for the scalability task; the broader study used 152 reference datasets from 24 platforms)
- Benchmark size grid: 40 sizes spanning 100-10,000 cells and 500-10,000 genes (Additional file 2 Table S8)

## Outputs

- scalability_results.csv containing per-method complexity classes, wall-clock execution times, peak memory consumption, and random forest / SCAM predicted-versus-actual Pearson correlations
- Per-method complexity class assignments (e.g. quadratic, super-quadratic, linear)

## How to apply

Load the seven reference datasets from the simmethods/simpipe environment and down- or up-sample each to the 40 benchmark sizes spanning 100-10,000 cells and 500-10,000 genes (Additional file 2 Table S8). Run each simulation method (e.g. dyntoy, SRTsim, scDesign2, scDesign3, ZINB-WaVE) on every resampled dataset via simpipe, recording wall-clock execution time and peak memory consumption per run. Fit the complexity GLM y ~ log(x)+sqrt(x)+x+x^2+x^3 to each method's time and memory measurements to assign a complexity class, and additionally train a random forest with 500 trees and a SCAM (shape-constrained additive model) regressor, computing predicted-versus-actual Pearson correlations. Expected patterns to confirm: dyntoy shows quadratic time complexity in the parameter estimation step and quadratic memory complexity in the data simulation step; SRTsim's gene-expression modeling and scDesign's simulation step show super-quadratic memory growth; scDesign2, scDesign3, and ZINB-WaVE are resource-heavy. Write complexity classes, resource measurements, and Pearson correlations to scalability_results.csv, and weigh scalability against accuracy since fast methods (Phenopath, Lun, Simple, MFA) may fail to generate realistic data.

## Related tools

- **simmethods** (Provides the 49 collected simulation methods (including dyntoy, ZINB-WaVE) and parameter estimation from real data that are benchmarked for scalability) — https://github.com/duohongrui/simmethods
- **simpipe** (Standard pipeline used to run each simulation method on resampled datasets and record execution time and peak memory) — https://github.com/duohongrui/simpipe
- **SCAM** (Shape-constrained additive model regressor used to predict execution time and peak memory and compute predicted-versus-actual Pearson correlations)
- **dyntoy** (Reference case with known quadratic time complexity in parameter estimation and quadratic memory complexity in data simulation, used to validate the scaling analysis) — https://github.com/dynverse/dyntoy
- **simpipe2docker** (Links the R environment to the simpipe Docker container so resource measurements run in a standardized environment) — https://github.com/duohongrui/simpipe2docker

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- dyntoy's measured execution time in the parameter estimation step fits a quadratic term (x^2) in the complexity GLM and its memory in the data simulation step is quadratic in cells/genes.
- SRTsim (gene-expression modeling) and scDesign (simulation step) show super-quadratic memory growth consistent with the reported patterns.
- Random forest (500 trees) and SCAM predicted-vs-actual Pearson correlations are reported for every method's time and memory measurements and written to scalability_results.csv.
- All 40 benchmark sizes (100-10,000 cells, 500-10,000 genes) are represented for each of the seven reference datasets, with wall-clock time and peak memory recorded for every run.
- Fast methods (Phenopath, Lun, Simple, MFA) show high scalability scores but are flagged as unable to generate realistic simulated data, confirming the accuracy-scalability trade-off.

## Limitations

- Users must consider trade-offs between method accuracy and scalability (or functionality); scalability alone cannot justify a method choice.
- Methods with heavy resource use (scDesign2, scDesign3, ZINB-WaVE) may be impractical at the largest benchmark sizes (10,000 cells x 10,000 genes).
- Execution errors can arise from failed parameter estimations and missing or infinite values, which must be handled when scaling methods to larger resampled datasets.

## Evidence

- [full_text] dyntoy quadratic scaling finding: [evidence span withheld — anchor status: not_found]
- [abstract] scalability-accuracy trade-off: "Phenopath, Lun, Simple, and MFA yield high scalability scores but they cannot generate realistic simulated data"
- [abstract] decision guidance: "Users should consider the trade-offs between method accuracy and scalability (or functionality) when making decisions"
- [methods] installation of benchmarking toolchain: "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [abstract] error cause: "execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations"
