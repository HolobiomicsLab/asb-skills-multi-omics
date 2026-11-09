---
name: peak-memory-monitoring
description: Use when benchmarking how a simulation method's runtime or peak memory
  scales with the number of cells (100-10,000) or genes (500-10,000), e.g. to flag
  quadratic or super-quadratic complexity before applying the method at full-dataset
  size.
when_to_use_negative:
- The goal is accuracy/usability assessment rather than scalability — this skill measures
  only execution time and peak memory, not fidelity of simulated data
- The method is not available through simmethods/simpipe (the pipeline depends on
  devtools-installed simmethods and simpipe)
- The input dataset cannot be legitimately down- or up-sampled to the benchmark sizes
  (e.g. trajectory or spatial structure that resampling would destroy)
tools:
- name: simpipe
  role: Standard pipeline that runs each simulation method and records wall-clock
    execution time and peak memory consumption per run
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Collection of the 49 simulation methods (e.g. dyntoy, SRTsim, scDesign2, scDesign3,
    ZINB-WaVE) being profiled
  repo: https://github.com/duohongrui/simmethods
- name: simpipe2docker
  role: Links the R environment to the simpipe Docker container so resource measurements
    are run in the standardized simmethods/simpipe environment
  repo: https://github.com/duohongrui/simpipe2docker
- name: SCAM
  role: Shape-constrained additive model regressor used to predict execution time
    and peak memory from dataset size
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:0030c6341ad2b83539d7044ef2f07353ab1d9d35ea7d86ea7397ac0664930177
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# peak-memory-monitoring

## Summary

Benchmark the scalability of scRNA-seq/SRT simulation methods by recording wall-clock execution time and peak memory consumption per run across resampled dataset sizes, then fitting complexity models to classify how each method scales with cells and genes.

## When to use

Use when benchmarking how a simulation method's runtime or peak memory scales with the number of cells (100-10,000) or genes (500-10,000), e.g. to flag quadratic or super-quadratic complexity before applying the method at full-dataset size.

## When NOT to use

- The goal is accuracy/usability assessment rather than scalability — this skill measures only execution time and peak memory, not fidelity of simulated data
- The method is not available through simmethods/simpipe (the pipeline depends on devtools-installed simmethods and simpipe)
- The input dataset cannot be legitimately down- or up-sampled to the benchmark sizes (e.g. trajectory or spatial structure that resampling would destroy)

## Inputs

- Seven reference scRNA-seq/SRT datasets loaded in the simmethods/simpipe R environment
- Resampled datasets at 40 benchmark sizes (100-10,000 cells; 500-10,000 genes, Additional file 2 Table S8)
- Simulation method calls via simmethods/simpipe for each of the evaluated methods (e.g. dyntoy, SRTsim, scDesign2, scDesign3, ZINB-WaVE)

## Outputs

- scalability_results.csv with complexity classes, wall-clock execution times, and peak memory measurements per method and dataset size
- Per-method complexity class from the GLM y ~ log(x)+sqrt(x)+x+x^2+x^3
- Predicted-versus-actual Pearson correlations for the 500-tree random forest and SCAM regressors

## How to apply

Load the seven reference datasets and down- or up-sample each to the 40 benchmark sizes spanning 100-10,000 cells and 500-10,000 genes (Additional file 2 Table S8). Run each simulation method on every resampled dataset via simpipe, recording wall-clock execution time and peak memory consumption for every run. Fit the complexity GLM y ~ log(x)+sqrt(x)+x+x^2+x^3 to the time and memory measurements of each method to assign its complexity class. Additionally train a random forest with 500 trees and a SCAM (shape-constrained additive model) regressor to predict execution time and peak memory, and compute predicted-versus-actual Pearson correlations to quantify predictive fidelity. Use the results to confirm known patterns such as dyntoy's quadratic time complexity in parameter estimation and quadratic memory complexity in the data simulation step.

## Related tools

- **simpipe** (Standard pipeline that runs each simulation method and records wall-clock execution time and peak memory consumption per run) — https://github.com/duohongrui/simpipe
- **simmethods** (Collection of the 49 simulation methods (e.g. dyntoy, SRTsim, scDesign2, scDesign3, ZINB-WaVE) being profiled) — https://github.com/duohongrui/simmethods
- **simpipe2docker** (Links the R environment to the simpipe Docker container so resource measurements are run in the standardized simmethods/simpipe environment) — https://github.com/duohongrui/simpipe2docker
- **SCAM** (Shape-constrained additive model regressor used to predict execution time and peak memory from dataset size)

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- The complexity GLM y ~ log(x)+sqrt(x)+x+x^2+x^3 fits the time and memory measurements and yields a complexity class per method
- Known patterns are reproduced: dyntoy quadratic time complexity (parameter estimation) and quadratic memory complexity (simulation step); super-quadratic memory growth for SRTsim (gene-expression modeling) and scDesign (simulation step)
- Predicted-versus-actual Pearson correlations are computed for both the 500-tree random forest and the SCAM regressor
- Every one of the 40 benchmark sizes (100-10,000 cells, 500-10,000 genes) has recorded time and peak-memory values for each method run
- All complexity classes, resource measurements, and Pearson correlations are written to scalability_results.csv

## Limitations

- Heavy resource use of scDesign2, scDesign3, and ZINB-WaVE limits their practicality at large cell/gene numbers
- High scalability does not imply usefulness: Phenopath, Lun, Simple, and MFA yield high scalability scores but cannot generate realistic simulated data
- Execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations, which can abort runs at extreme sizes

## Evidence

- [abstract] Systematic evaluation of simulation methods in terms of accuracy, functionality, scalability, and usability: "We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability"
- [abstract] Scalability scores do not equal realism: "Phenopath, Lun, Simple, and MFA yield high scalability scores but they cannot generate realistic simulated data"
- [methods] simpipe installed via devtools: "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [readme] simpipe scope: "The goal of simpipe is to establish a standard pipeline to estimating
parameters from real datasets, simulating single-cell RNA-seq datasets
and evaluating the simulated data"
