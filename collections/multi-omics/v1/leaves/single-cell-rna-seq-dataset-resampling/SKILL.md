---
name: single-cell-rna-seq-dataset-resampling
description: 'Use when benchmarking the scalability of simulation methods: you need
  datasets spanning a controlled size grid (e.g., 40 sizes from 100–10,000 cells and
  500–10,000 genes) to record wall-clock time and peak memory per run.'
when_to_use_negative:
- When the goal is accuracy/usability benchmarking of simulation methods rather than
  scalability (time and memory) profiling
- When the reference dataset cannot be validly down- or up-sampled to the target size
  grid (e.g., too few cells/genes to reach the required benchmark dimensions)
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: simpipe
  role: Standard pipeline used to run each simulation method on every resampled dataset
    while recording execution time and peak memory
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Collection of 49 simulation methods from which the benchmarked simulators
    are run on the resampled datasets
  repo: https://github.com/duohongrui/simmethods
- name: SCAM
  role: Shape-constrained additive model regressor used to predict execution time
    and peak memory from the scalability measurements
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:dbbb3b6027982e486b565479e2692695f8eba54850ffaeaec79a9edca45b5e25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simpipe
  tools:
  - SCAM
  - SRTsim
  - ZINB-WaVE
  - dyntoy
  - scDesign2
  - scDesign3
  - simmethods
  - simpipe
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# single-cell-rna-seq-dataset-resampling

## Summary

Resamples reference scRNA-seq/SRT datasets down- or up-sampled to a grid of benchmark sizes (100–10,000 cells, 500–10,000 genes) so that simulation-method execution time and peak memory can be measured across scales and fitted to a complexity GLM.

## When to use

Use when benchmarking the scalability of simulation methods: you need datasets spanning a controlled size grid (e.g., 40 sizes from 100–10,000 cells and 500–10,000 genes) to record wall-clock time and peak memory per run.

## When NOT to use

- When the goal is accuracy/usability benchmarking of simulation methods rather than scalability (time and memory) profiling
- When the reference dataset cannot be validly down- or up-sampled to the target size grid (e.g., too few cells/genes to reach the required benchmark dimensions)

## Inputs

- Seven reference scRNA-seq/SRT datasets loaded in the simmethods/simpipe R environment
- Benchmark size grid of 40 sizes spanning 100–10,000 cells and 500–10,000 genes (Additional file 2 Table S8)
- Simulation methods to be benchmarked (e.g., dyntoy, SRTsim, scDesign2, scDesign3, ZINB-WaVE)

## Outputs

- scalability_results.csv with per-run wall-clock execution time, peak memory consumption, fitted complexity classes, and predicted-versus-actual Pearson correlations
- Per-method complexity classes from the GLM y ~ log(x)+sqrt(x)+x+x^2+x^3
- Random forest (500 trees) and SCAM predictions of execution time and peak memory

## How to apply

Load the seven reference datasets in the simmethods/simpipe environment and down- or up-sample each to the 40 benchmark sizes spanning 100–10,000 cells and 500–10,000 genes (Additional file 2 Table S8). Run each simulation method on every resampled dataset via simpipe, recording wall-clock execution time and peak memory consumption for each run. Fit the complexity GLM y ~ log(x)+sqrt(x)+x+x^2+x^3 to the time and memory measurements per method to assign its complexity class (e.g., dyntoy shows quadratic time complexity in parameter estimation and quadratic memory complexity in data simulation). Additionally train a random forest with 500 trees and a SCAM (shape-constrained additive model) regressor to predict execution time and peak memory, and compute predicted-versus-actual Pearson correlations. Write all complexity classes, resource measurements, and correlations to scalability_results.csv.

## Related tools

- **simpipe** (Standard pipeline used to run each simulation method on every resampled dataset while recording execution time and peak memory) — https://github.com/duohongrui/simpipe
- **simmethods** (Collection of 49 simulation methods from which the benchmarked simulators are run on the resampled datasets) — https://github.com/duohongrui/simmethods
- **SCAM** (Shape-constrained additive model regressor used to predict execution time and peak memory from the scalability measurements)

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- scalability_results.csv is produced with per-run execution time, peak memory, complexity classes, and Pearson correlations
- Each method's complexity class is consistent with the GLM y ~ log(x)+sqrt(x)+x+x^2+x^3 fit over the 40 benchmark sizes
- Known scalability patterns are reproduced: dyntoy quadratic time complexity (parameter estimation) and quadratic memory complexity (data simulation)
- Random forest (500 trees) and SCAM predicted-versus-actual Pearson correlations are computed for both time and memory
- All 40 sizes in the 100–10,000 cell and 500–10,000 gene grid (Table S8) are covered for each of the seven reference datasets

## Limitations

- Some methods have heavy resource demands at large sizes — scDesign2, scDesign3, and ZINB-WaVE are resource-intensive, and SRTsim (gene-expression modeling) and scDesign (simulation step) show super-quadratic memory growth — so large-grid runs may fail or be very slow
- Execution errors mainly arise from failed parameter estimations and the appearance of missing or infinite values in calculations during resampling or simulation
- High scalability does not imply realistic output: Phenopath, Lun, Simple, and MFA yield high scalability scores but cannot generate realistic simulated data

## Evidence

- [abstract] 49 simulation methods evaluated using 152 reference datasets from 24 platforms on accuracy, functionality, scalability, and usability: "We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability using 152 reference datasets derived from 24"
- [readme] simpipe purpose: "Simpipe provides the standard pipeline of estimating parameters from read data, simulating and evaluating simulated data."
- [methods] installing simmethods and simpipe: "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [abstract] scalability versus realism trade-off: "Phenopath, Lun, Simple, and MFA yield high scalability scores but they cannot generate realistic simulated data"
