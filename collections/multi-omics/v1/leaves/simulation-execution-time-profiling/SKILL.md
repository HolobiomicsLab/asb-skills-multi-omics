---
name: simulation-execution-time-profiling
description: Use when benchmarking how execution time and memory consumption of scRNA-seq/SRT
  simulators (e.g., dyntoy, SRTsim, scDesign, scDesign2, scDesign3, ZINB-WaVE) scale
  with cell or gene numbers.
when_to_use_negative:
- When the goal is accuracy/usability benchmarking of simulators rather than scalability
  profiling — this skill measures only time and memory scaling
- When datasets cannot be down- or up-sampled to the 100-10,000 cell / 500-10,000
  gene range required by the benchmark design
- When a simulation method cannot run at small sizes (execution errors from failed
  parameter estimation or missing/infinite values), as its scaling curve cannot be
  fit reliably
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: simpipe
  role: Standard pipeline that executes each simulation method on resampled datasets
    while recording wall-clock execution time and peak memory consumption
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Collection of 49 simulation methods (including dyntoy, scDesign2, scDesign3,
    ZINB-WaVE) whose scalability is profiled
  repo: https://github.com/duohongrui/simmethods
- name: SCAM
  role: Shape-constrained additive model regressor used to predict execution time
    and peak memory alongside the 500-tree random forest
- name: simpipe2docker
  role: Links the R environment and the Docker container so simulations can be run
    reproducibly in the simpipe Docker image
  repo: https://github.com/duohongrui/simpipe2docker
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:6c7b0553a302c18c6b5ef8747d9ba47a64ed31e9422b9a34e1a29884f028aec5
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
  - simpipe2docker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# simulation-execution-time-profiling

## Summary

Profiles the computational scalability of scRNA-seq/SRT simulation methods by running each simulator on datasets resampled across 40 benchmark sizes (100-10,000 cells, 500-10,000 genes), recording wall-clock time and peak memory, and fitting a complexity GLM to assign each method a complexity class.

## When to use

Use when benchmarking how execution time and memory consumption of scRNA-seq/SRT simulators (e.g., dyntoy, SRTsim, scDesign, scDesign2, scDesign3, ZINB-WaVE) scale with cell or gene numbers.

## When NOT to use

- When the goal is accuracy/usability benchmarking of simulators rather than scalability profiling — this skill measures only time and memory scaling
- When datasets cannot be down- or up-sampled to the 100-10,000 cell / 500-10,000 gene range required by the benchmark design
- When a simulation method cannot run at small sizes (execution errors from failed parameter estimation or missing/infinite values), as its scaling curve cannot be fit reliably

## Inputs

- Reference single-cell or SRT datasets loaded in the simmethods/simpipe environment (seven reference datasets)
- Benchmark size grid from Additional file 2 Table S8 (40 sizes, 100-10,000 cells; 500-10,000 genes)

## Outputs

- scalability_results.csv with complexity classes, wall-clock execution times, peak memory measurements, and predicted-versus-actual Pearson correlations
- Per-method complexity class assignments (e.g., quadratic for dyntoy, super-quadratic for SRTsim/scDesign memory growth)

## How to apply

Load the seven reference datasets in the simmethods/simpipe environment and down- or up-sample each to the 40 benchmark sizes spanning 100-10,000 cells and 500-10,000 genes (Additional file 2 Table S8). Run each simulation method on every resampled dataset via simpipe, recording wall-clock execution time and peak memory per run. Fit the complexity GLM y ~ log(x)+sqrt(x)+x+x^2+x^3 to the time and memory measurements of each method to assign its complexity class. Additionally train a random forest with 500 trees and a SCAM (shape-constrained additive model) regressor to predict execution time and peak memory, computing predicted-versus-actual Pearson correlations. Confirm known patterns such as dyntoy's quadratic time complexity (in parameter estimation) and quadratic memory complexity (in data simulation), super-quadratic memory growth of SRTsim (gene-expression modeling) and scDesign (simulation step), and heavy resource use of scDesign2, scDesign3, and ZINB-WaVE.

## Related tools

- **simpipe** (Standard pipeline that executes each simulation method on resampled datasets while recording wall-clock execution time and peak memory consumption) — https://github.com/duohongrui/simpipe
- **simmethods** (Collection of 49 simulation methods (including dyntoy, scDesign2, scDesign3, ZINB-WaVE) whose scalability is profiled) — https://github.com/duohongrui/simmethods
- **SCAM** (Shape-constrained additive model regressor used to predict execution time and peak memory alongside the 500-tree random forest)
- **simpipe2docker** (Links the R environment and the Docker container so simulations can be run reproducibly in the simpipe Docker image) — https://github.com/duohongrui/simpipe2docker

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- scalability_results.csv contains complexity classes, time/memory measurements, and Pearson correlations for every method-dataset combination
- dyntoy reproduces quadratic time complexity in the parameter estimation step and quadratic memory complexity in the data simulation step
- SRTsim (gene-expression modeling) and scDesign (simulation step) show super-quadratic memory growth; scDesign2, scDesign3, and ZINB-WaVE show heavy resource use
- Predicted-versus-actual Pearson correlations from the random forest and SCAM regressors are reported for execution time and peak memory predictions
- All 40 benchmark sizes (100-10,000 cells, 500-10,000 genes) from Table S8 are covered for each resampled dataset

## Limitations

- Highly scalable methods (e.g., Phenopath, Lun, Simple, MFA) may still produce unrealistic simulated data, so scalability must be weighed against accuracy
- Execution errors — mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations — prevent profiling some methods at some sizes
- No method performs best on all criteria; users should consider trade-offs between method accuracy and scalability (or functionality) when interpreting the profiled results

## Evidence

- [results] dyntoy quadratic time and memory; workflow on resampled datasets: [evidence span withheld — anchor status: not_found]
- [readme] simpipe as the standard pipeline: "Simpipe provides the standard pipeline of estimating parameters from read data, simulating and evaluating simulated data."
- [abstract] scope of the systematic evaluation: "We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability"
- [abstract] scalable methods produce unrealistic data: "Phenopath, Lun, Simple, and MFA yield high scalability scores but they cannot generate realistic simulated data"
- [methods] installation command from methods: "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
