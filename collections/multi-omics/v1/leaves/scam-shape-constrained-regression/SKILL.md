---
name: scam-shape-constrained-regression
description: Use when benchmark measurements (runtime, memory) across increasing cell/gene
  counts need a shape-constrained regression fit to assign complexity classes and
  predict resource consumption of simulation methods.
when_to_use_negative:
- The goal is accuracy/fidelity benchmarking of simulated data rather than scalability
  prediction of runtime and memory
- Measurements are not ordered along a size axis (e.g., single fixed dataset size),
  so shape-constrained fitting over increasing cells/genes is meaningless
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: simpipe
  role: Standard pipeline used to run each simulation method on resampled datasets
    and record wall-clock execution time and peak memory that the SCAM regressor is
    trained on
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Collection of the 49 simulation methods (e.g., dyntoy, ZINB-WaVE) whose scalability
    measurements feed the SCAM regression
  repo: https://github.com/duohongrui/simmethods
- name: SCAM
  role: Shape-constrained additive model regressor trained to predict execution time
    and peak memory
- name: dyntoy
  role: Simulation method whose measured quadratic time (parameter estimation) and
    quadratic memory (data simulation) complexity the SCAM/complexity-class analysis
    must confirm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:7b96e71b143a0b0228f6a9a5e3e6749a2c275289c75e46650470cd17dcad1bdc
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# scam-shape-constrained-regression

## Summary

Fits a SCAM (shape-constrained additive model) regressor to benchmarking measurements (e.g., wall-clock execution time and peak memory of simulation methods across 40 resampled dataset sizes) so that predicted resource use respects monotonic/shape constraints, and validates it via predicted-versus-actual Pearson correlation alongside a 500-tree random forest.

## When to use

Use when benchmark measurements (runtime, memory) across increasing cell/gene counts need a shape-constrained regression fit to assign complexity classes and predict resource consumption of simulation methods.

## When NOT to use

- The goal is accuracy/fidelity benchmarking of simulated data rather than scalability prediction of runtime and memory
- Measurements are not ordered along a size axis (e.g., single fixed dataset size), so shape-constrained fitting over increasing cells/genes is meaningless

## Inputs

- Wall-clock execution time and peak memory measurements per simulation method run via simpipe on 40 resampled dataset sizes (100-10,000 cells; 500-10,000 genes)
- Seven reference scRNA-seq/SRT datasets resampled per Additional file 2 Table S8
- GLM complexity fits (y ~ log(x)+sqrt(x)+x+x^2+x^3) used to assign complexity classes

## Outputs

- Predicted execution time and peak memory per method from the SCAM regressor
- Predicted-versus-actual Pearson correlations for the SCAM and 500-tree random forest predictors
- scalability_results.csv with complexity classes, resource measurements, and Pearson correlations

## How to apply

In the scalability evaluation of scRNA-seq/SRT simulation methods, first collect wall-clock execution time and peak memory for each method (e.g., dyntoy, SRTsim, scDesign2, scDesign3, ZINB-WaVE) run via simpipe on datasets down- or up-sampled to 40 benchmark sizes spanning 100-10,000 cells and 500-10,000 genes. Fit the complexity GLM y ~ log(x)+sqrt(x)+x+x^2+x^3 to these measurements and assign each method a complexity class, then train a SCAM regressor (alongside a random forest with 500 trees) to predict execution time and peak memory from dataset size. The shape constraints prevent non-monotonic fits that would contradict physically plausible resource growth (e.g., dyntoy's quadratic time complexity in parameter estimation and quadratic memory complexity in data simulation). Judge correctness by computing predicted-versus-actual Pearson correlations and by whether recovered classes match known patterns such as super-quadratic memory growth of SRTsim (gene-expression modeling) and scDesign (simulation step). Write complexity classes, resource measurements, and Pearson correlations to scalability_results.csv.

## Related tools

- **simpipe** (Standard pipeline used to run each simulation method on resampled datasets and record wall-clock execution time and peak memory that the SCAM regressor is trained on) — https://github.com/duohongrui/simpipe
- **simmethods** (Collection of the 49 simulation methods (e.g., dyntoy, ZINB-WaVE) whose scalability measurements feed the SCAM regression) — https://github.com/duohongrui/simmethods
- **SCAM** (Shape-constrained additive model regressor trained to predict execution time and peak memory)
- **dyntoy** (Simulation method whose measured quadratic time (parameter estimation) and quadratic memory (data simulation) complexity the SCAM/complexity-class analysis must confirm)

## Evaluation signals

- Predicted-versus-actual Pearson correlations are computed for the SCAM and random-forest resource predictions and written to scalability_results.csv
- Recovered complexity classes reproduce known patterns: dyntoy quadratic time complexity in parameter estimation and quadratic memory complexity in data simulation
- Super-quadratic memory growth is detected for SRTsim (gene-expression modeling) and scDesign (simulation step)
- All 40 benchmark sizes span 100-10,000 cells and 500-10,000 genes per Additional file 2 Table S8
- Heavy resource use is flagged for scDesign2, scDesign3, and ZINB-WaVE

## Limitations

- Phenopath, Lun, Simple, and MFA yield high scalability scores but cannot generate realistic simulated data, so scalability results must be interpreted jointly with accuracy
- Users should consider the trade-offs between method accuracy and scalability (or functionality) when making decisions
- Some methods incur execution errors from failed parameter estimations or missing/infinite values in calculations, leaving gaps in the measurements to fit

## Evidence

- [methods] workflow step 4: [evidence span withheld — anchor status: not_found]
- [methods] workflow step 3: [evidence span withheld — anchor status: not_found]
- [abstract] scalability vs realism caveat: "Phenopath, Lun, Simple, and MFA yield high scalability scores but they cannot generate realistic simulated data"
- [abstract] simpipe citation: "a standard pipeline Simpipe ( https://github.com/duohongrui/simpipe ; https://doi.org/10.5281/zenodo.11178409 )"
- [readme] installation: "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
