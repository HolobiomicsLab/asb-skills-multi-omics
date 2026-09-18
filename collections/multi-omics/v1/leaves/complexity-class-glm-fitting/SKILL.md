---
name: complexity-class-glm-fitting
description: 'Use when benchmarking simulation methods'' scalability: you have execution-time
  and peak-memory measurements across 40 dataset sizes (100-10,000 cells, 500-10,000
  genes) and need to assign a complexity class per method.'
when_to_use_negative:
- Evaluating simulation accuracy/fidelity of the simulated data — this skill only
  addresses resource scaling, and the article notes high-scalability methods like
  Phenopath, Lun, Simple, and MFA 'cannot generate realistic simulated data'
- The input is a single run of one method at one dataset size — complexity-class fitting
  requires measurements across a range of sizes (100-10,000 cells and 500-10,000 genes)
edam_operation: http://edamontology.org/operation_3659
tools:
- name: simpipe
  role: Runs each simulation method on every resampled dataset, recording wall-clock
    execution time and peak memory consumption for each run
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Provides the collection of 49 simulation methods whose time and memory measurements
    are fitted to the complexity GLM
  repo: https://github.com/duohongrui/simmethods
- name: dyntoy
  role: Method confirmed to exhibit quadratic time complexity in parameter estimation
    and quadratic memory complexity in data simulation
  repo: https://github.com/dynverse/dyntoy
- name: SCAM
  role: Shape-constrained additive model regressor used to predict execution time
    and peak memory
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:85912447d4a0d02a8e7601f7ecfed2d455a9c0b9a7d896d4e984e2b4e61a24af
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# complexity-class-glm-fitting

## Summary

Fits a polynomial-feature GLM (y ~ log(x)+sqrt(x)+x+x^2+x^3) to wall-clock execution time and peak memory measurements collected across resampled dataset sizes, in order to assign each scRNA-seq/SRT simulation method a computational complexity class (e.g. dyntoy's quadratic scaling).

## When to use

Use when benchmarking simulation methods' scalability: you have execution-time and peak-memory measurements across 40 dataset sizes (100-10,000 cells, 500-10,000 genes) and need to assign a complexity class per method.

## When NOT to use

- Evaluating simulation accuracy/fidelity of the simulated data — this skill only addresses resource scaling, and the article notes high-scalability methods like Phenopath, Lun, Simple, and MFA 'cannot generate realistic simulated data'
- The input is a single run of one method at one dataset size — complexity-class fitting requires measurements across a range of sizes (100-10,000 cells and 500-10,000 genes)

## Inputs

- Wall-clock execution time measurements per method per benchmark size (40 sizes, 100-10,000 cells; 500-10,000 genes)
- Peak memory consumption measurements per method per benchmark size
- Seven reference datasets from the simmethods/simpipe environment, down- or up-sampled per Additional file 2 Table S8

## Outputs

- scalability_results.csv containing complexity classes, resource measurements, and predicted-versus-actual Pearson correlations per method
- Complexity-class assignment per method (e.g. quadratic time complexity for dyntoy's parameter estimation step)
- Random forest (500 trees) and SCAM regressor predictions with Pearson correlations

## How to apply

First resample seven reference datasets to 40 benchmark sizes spanning 100-10,000 cells and 500-10,000 genes, and run each simulation method via simpipe while recording wall-clock execution time and peak memory per run. Fit the complexity GLM y ~ log(x)+sqrt(x)+x+x^2+x^3 to the time and memory measurements of each method and assign its complexity class from the dominant fitted term. As a cross-check, train a random forest with 500 trees and a SCAM (shape-constrained additive model) regressor to predict execution time and peak memory, then compute predicted-versus-actual Pearson correlations to quantify predictive reliability. The rationale is that term significance/monotone shape in the GLM distinguishes constant, linear, quadratic, and super-quadratic growth — e.g. confirming dyntoy's quadratic time complexity in parameter estimation and quadratic memory complexity in data simulation, plus super-quadratic memory growth of SRTsim and scDesign in their gene-expression modeling/simulation steps. Write all complexity classes, resource measurements, and Pearson correlations to scalability_results.csv.

## Related tools

- **simpipe** (Runs each simulation method on every resampled dataset, recording wall-clock execution time and peak memory consumption for each run) — https://github.com/duohongrui/simpipe
- **simmethods** (Provides the collection of 49 simulation methods whose time and memory measurements are fitted to the complexity GLM) — https://github.com/duohongrui/simmethods
- **dyntoy** (Method confirmed to exhibit quadratic time complexity in parameter estimation and quadratic memory complexity in data simulation) — https://github.com/dynverse/dyntoy
- **SCAM** (Shape-constrained additive model regressor used to predict execution time and peak memory)

## Examples

```
model <- lm(time ~ log(n) + sqrt(n) + n + I(n^2) + I(n^3), data = scalability_results)
```

## Evaluation signals

- The GLM formula used is exactly y ~ log(x)+sqrt(x)+x+x^2+x^3 fitted separately to time and memory measurements of each method
- Fitted complexity classes reproduce known patterns: dyntoy quadratic time (parameter estimation) and quadratic memory (data simulation); SRTsim and scDesign super-quadratic memory growth; heavy resource use of scDesign2, scDesign3, and ZINB-WaVE
- Predicted-versus-actual Pearson correlations are computed for the random forest (500 trees) and SCAM regressors and reported alongside complexity classes in scalability_results.csv
- Measurements cover all 40 benchmark sizes spanning 100-10,000 cells and 500-10,000 genes derived from seven reference datasets

## Limitations

- Users should consider the trade-offs between method accuracy and scalability (or functionality) when making decisions — scalability alone does not indicate a method is fit for purpose
- Methods with high scalability scores such as Phenopath, Lun, Simple, and MFA cannot generate realistic simulated data
- Some methods exhibit heavy resource use (scDesign2, scDesign3, ZINB-WaVE) and super-quadratic memory growth, which can prevent scaling to large cell/gene counts

## Evidence

- [methods] GLM formula grounds the model: [evidence span withheld — anchor status: not_found]
- [results] dyntoy scalability finding: [evidence span withheld — anchor status: not_found]
- [methods] Random forest and SCAM cross-check: [evidence span withheld — anchor status: not_found]
- [abstract] Trade-off caveat from abstract: "Users should consider the trade-offs between method accuracy and scalability (or functionality) when making decisions"
- [abstract] Scalability-scores caveat: "Phenopath, Lun, Simple, and MFA yield high scalability scores but they cannot generate realistic simulated data"
