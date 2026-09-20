---
name: benchmark-accuracy-scoring
description: Use when a systematic comparison of single-cell or spatial transcriptomics
  simulation methods is needed and per-method accuracy must be quantified against
  real reference data.
when_to_use_negative:
- The goal is scalability or usability benchmarking rather than fidelity — Phenopath,
  Lun, Simple, and MFA yield high scalability scores but cannot generate realistic
  simulated data, so scalability alone must not be read as accuracy.
- Only one simulation method needs parameter estimation and dataset generation (use
  simmethods directly) rather than a comparative benchmark against real reference
  data.
edam_operation: http://edamontology.org/operation_2945
tools:
- name: simpipe
  role: Standard pipeline that wraps each simulation method, estimates parameters
    from real data, simulates datasets, and evaluates simulated data from general
    and functional perspectives (8 accuracy metrics over 15 data properties).
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Collection of the 49 simulation methods for single-cell and spatial transcriptome
    data that simpipe executes for the accuracy benchmark.
  repo: https://github.com/duohongrui/simmethods
- name: simpipe2docker
  role: Links the R environment and the simpipe Docker container so simulations and
    evaluations can run inside the container.
  repo: https://github.com/duohongrui/simpipe2docker
- name: tislingshot
  role: Installed first when using simpipe to simulate datasets with trajectory information,
    extending accuracy benchmarking to trajectory scenarios.
  repo: https://github.com/dynverse/ti_slingshot/package/
- name: devtools
  role: Used to install simmethods and simpipe from GitHub (devtools::install_github).
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:29f4e6daf39317d34dd75d4f8dc0917b39c0ae705dc88207e1057176a75a78d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simpipe
  tools:
  - SRTsim
  - devtools
  - simmethods
  - simpipe
  - simpipe2docker
  - tislingshot
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# benchmark-accuracy-scoring

## Summary

Benchmark accuracy scoring for scRNA-seq/SRT simulation methods: run each simulator on real reference datasets via simpipe, compare simulated data against the real reference across 15 data properties with 8 accuracy metrics, and aggregate into a per-method accuracy score.

## When to use

Use when a systematic comparison of single-cell or spatial transcriptomics simulation methods is needed and per-method accuracy must be quantified against real reference data.

## When NOT to use

- The goal is scalability or usability benchmarking rather than fidelity — Phenopath, Lun, Simple, and MFA yield high scalability scores but cannot generate realistic simulated data, so scalability alone must not be read as accuracy.
- Only one simulation method needs parameter estimation and dataset generation (use simmethods directly) rather than a comparative benchmark against real reference data.

## Inputs

- Reference scRNA-seq and spatially resolved transcriptomics datasets (152 datasets from 24 platforms; Additional file 2, Tables S1-S3, DOI 10.1186/s13059-024-03290-y)
- Simulation method wrappers from the simmethods package (49 methods, e.g., SRTsim, scDesign3, ZINB-WaVE, scDesign2, Lun, SPARSim, scDesign3-tree)
- simpipe evaluation wrappers implementing 8 accuracy metrics over 15 data properties

## Outputs

- accuracy_scores.csv with a single aggregated accuracy score per simulation method
- Per-metric comparison results for each simulated dataset vs its real reference across 15 data properties
- Accuracy ranking, stratified overall and by scRNA-seq vs SRT technique

## How to apply

Install simmethods and simpipe from GitHub with devtools (and tislingshot from dynverse/ti_slingshot when simulating datasets with trajectory information), or run everything through the simpipe Docker image linked to R via simpipe2docker. Execute each of the 49 simulation methods on the applicable reference scRNA-seq/SRT datasets using the simpipe wrappers. For each simulated dataset, compare it against its real reference across 15 data properties using the 8 accuracy metrics implemented in simpipe, then aggregate metric results into a single accuracy score per method, both overall and stratified by scRNA-seq vs SRT technique. Write results to accuracy_scores.csv and check the ranking against reported top performers (SRTsim 0.84, scDesign3-tree 0.78, ZINB-WaVE 0.77, scDesign3 0.76, scDesign2 0.74; Class 2 mean 0.60).

## Related tools

- **simpipe** (Standard pipeline that wraps each simulation method, estimates parameters from real data, simulates datasets, and evaluates simulated data from general and functional perspectives (8 accuracy metrics over 15 data properties).) — https://github.com/duohongrui/simpipe
- **simmethods** (Collection of the 49 simulation methods for single-cell and spatial transcriptome data that simpipe executes for the accuracy benchmark.) — https://github.com/duohongrui/simmethods
- **simpipe2docker** (Links the R environment and the simpipe Docker container so simulations and evaluations can run inside the container.) — https://github.com/duohongrui/simpipe2docker
- **tislingshot** (Installed first when using simpipe to simulate datasets with trajectory information, extending accuracy benchmarking to trajectory scenarios.) — https://github.com/dynverse/ti_slingshot/package/
- **devtools** (Used to install simmethods and simpipe from GitHub (devtools::install_github).)

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- Reproduced top-5 accuracy ranking: SRTsim (0.84), scDesign3-tree (0.78), ZINB-WaVE (0.77), scDesign3 (0.76), scDesign2 (0.74).
- Class 2 methods attain a mean accuracy score around 0.60, consistent with the reported benchmark.
- accuracy_scores.csv is produced with one aggregated accuracy score per method, both overall and stratified by scRNA-seq vs SRT technique.
- Each simulated dataset has been compared against its real reference across all 15 data properties using all 8 accuracy metrics implemented in simpipe.
- Methods claimed as top performers match the article's abstract claim that 'SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across various platforms'.

## Limitations

- No method performs best on all criteria; a good-yet-not-the-best method is recommended if it solves problems effectively and reasonably, so users should consider trade-offs between method accuracy and scalability (or functionality).
- Execution errors are mainly caused by failed parameter estimations and the appearance of missing or infinite values in calculations.
- Simulation scenarios requiring trajectory information need tislingshot installed beforehand; alternatively the simpipe Docker image must be pulled and linked via simpipe2docker.

## Evidence

- [abstract] Top performers and scores: "SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across various platforms"
- [abstract] Scope of benchmark: "We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability using 152 reference datasets derived from 24"
- [abstract] Trade-off caveat: "Users should consider the trade-offs between method accuracy and scalability (or functionality) when making decisions"
- [readme] Installation commands: "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [readme] Trajectory prerequisite: "if you want to use simpipe to simulate datasets with **trajectory information**, please install **tislingshot** first"
- [readme] simpipe purpose: "The goal of simpipe is to establish a standard pipeline to estimating parameters from real datasets, simulating single-cell RNA-seq datasets and evaluating the simulated data"
- [abstract] Failure mode: "execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations"
