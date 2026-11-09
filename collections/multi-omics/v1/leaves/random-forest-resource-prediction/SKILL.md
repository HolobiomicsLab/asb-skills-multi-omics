---
name: random-forest-resource-prediction
description: Use when you need to predict or characterize the computational cost (execution
  time, peak memory) of simulation methods across resampled dataset sizes from 100-10,000
  cells and 500-10,000 genes.
when_to_use_negative:
- When the goal is evaluating the accuracy or fidelity of simulated data rather than
  its computational cost — that requires the accuracy evaluation workflow, not resource
  prediction
- When the simulation method is not runnable through simpipe/simpipe2docker and therefore
  execution time and peak memory cannot be recorded per run
- When datasets fall outside the benchmark size range (e.g., >10,000 cells or <500
  genes), where the fitted complexity models were not validated
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: simpipe
  role: Standard pipeline that executes each simulation method on resampled datasets
    and records wall-clock execution time and peak memory consumption
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Collection of the 49 simulation methods whose scalability is measured and
    predicted
  repo: https://github.com/duohongrui/simmethods
- name: SCAM
  role: Shape-constrained additive model regressor used alongside the random forest
    to predict execution time and peak memory
- name: dyntoy
  role: Simulation method whose known quadratic time complexity in parameter estimation
    and quadratic memory complexity in data simulation serve as validation targets
  repo: https://github.com/dynverse/dyntoy
- name: simpipe2docker
  role: Links the R environment to the simpipe Docker container for reproducible resource
    measurement
  repo: https://github.com/duohongrui/simpipe2docker
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:61f1bb6faf55a9981cdc54b88eaa756b114b32b07550166c8946627d421951a5
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# Random forest resource prediction

## Summary

Trains a random forest with 500 trees and a SCAM (shape-constrained additive model) regressor to predict wall-clock execution time and peak memory consumption of scRNA-seq/SRT simulation methods, validating predictions against measured scalability data from the simpipe benchmark of 49 simulation methods.

## When to use

Use when you need to predict or characterize the computational cost (execution time, peak memory) of simulation methods across resampled dataset sizes from 100-10,000 cells and 500-10,000 genes.

## When NOT to use

- When the goal is evaluating the accuracy or fidelity of simulated data rather than its computational cost — that requires the accuracy evaluation workflow, not resource prediction
- When the simulation method is not runnable through simpipe/simpipe2docker and therefore execution time and peak memory cannot be recorded per run
- When datasets fall outside the benchmark size range (e.g., >10,000 cells or <500 genes), where the fitted complexity models were not validated

## Inputs

- Seven reference scRNA-seq/SRT datasets loaded in the simmethods/simpipe environment
- Resampled datasets spanning 40 benchmark sizes (100-10,000 cells; 500-10,000 genes, Additional file 2 Table S8)
- Measured wall-clock execution time and peak memory consumption for each method run

## Outputs

- scalability_results.csv containing complexity classes, resource measurements, and Pearson correlations
- Per-method complexity class assignments (e.g., dyntoy quadratic time complexity)
- Predicted-versus-actual Pearson correlations for execution time and peak memory

## How to apply

First run each simulation method on resampled datasets via simpipe, recording wall-clock execution time and peak memory for every run across the 40 benchmark sizes derived from seven reference datasets. Fit a complexity GLM y ~ log(x)+sqrt(x)+x+x^2+x^3 to the time and memory measurements of each method to assign its complexity class. Train a random forest with 500 trees and a SCAM regressor to predict execution time and peak memory from the resampling features. Compute predicted-versus-actual Pearson correlations to quantify prediction quality, and write all complexity classes, resource measurements, and correlations to scalability_results.csv. The rationale is that scalability trade-offs matter because fast methods (Phenopath, Lun, Simple, MFA) may fail to generate realistic data, so resource prediction guides method selection.

## Related tools

- **simpipe** (Standard pipeline that executes each simulation method on resampled datasets and records wall-clock execution time and peak memory consumption) — https://github.com/duohongrui/simpipe
- **simmethods** (Collection of the 49 simulation methods whose scalability is measured and predicted) — https://github.com/duohongrui/simmethods
- **SCAM** (Shape-constrained additive model regressor used alongside the random forest to predict execution time and peak memory)
- **dyntoy** (Simulation method whose known quadratic time complexity in parameter estimation and quadratic memory complexity in data simulation serve as validation targets) — https://github.com/dynverse/dyntoy
- **simpipe2docker** (Links the R environment to the simpipe Docker container for reproducible resource measurement) — https://github.com/duohongrui/simpipe2docker

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- Predicted-versus-actual Pearson correlation for execution time and peak memory is computed and recorded in scalability_results.csv
- dyntoy's quadratic time complexity (parameter estimation) and quadratic memory complexity (data simulation) are recovered as expected
- Super-quadratic memory growth is confirmed for SRTsim (gene-expression modeling) and scDesign (simulation step)
- Heavy resource use is detected for scDesign2, scDesign3, and ZINB-WaVE as known patterns
- All 40 benchmark sizes (100-10,000 cells, 500-10,000 genes) are covered by recorded time/memory measurements for each method

## Limitations

- Highly scalable methods such as Phenopath, Lun, Simple, and MFA yield high scalability scores but cannot generate realistic simulated data, so predicted low resource cost alone does not imply a good method
- Resource predictions are tied to the resampling range of 100-10,000 cells and 500-10,000 genes; extrapolation beyond these sizes is unvalidated
- No method performs best on all criteria, so execution-time predictions must be weighed against accuracy, functionality, and usability

## Evidence

- [abstract] Phenopath, Lun, Simple, and MFA yield high scalability scores but they cannot generate realistic simulated data: "Phenopath, Lun, Simple, and MFA yield high scalability scores but they cannot generate realistic simulated data"
- [abstract] We systematically evaluated 49 simulation methods in terms of accuracy, functionality, scalability, and usability using 152 reference datasets derived from 24: "We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability using 152 reference datasets derived from 24"
- [methods] devtools::install_github("duohongrui/simmethods") and devtools::install_github("duohongrui/simpipe"): "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [supplementary] Fit the complexity GLM y ~ log(x)+sqrt(x)+x+x^2+x^3 to the time and memory measurements of each method and assign its complexity class: [evidence span withheld — anchor status: not_found]
- [readme] if you want to use simpipe to simulate datasets with trajectory information, please install tislingshot first: "if you want to use simpipe to simulate datasets with **trajectory
information**, please install **tislingshot** first"
