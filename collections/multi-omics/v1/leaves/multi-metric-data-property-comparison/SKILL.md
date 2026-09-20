---
name: multi-metric-data-property-comparison
description: Use when you have paired simulated and real reference single-cell RNA-seq
  or spatially resolved transcriptomics datasets and need to quantify how faithfully
  the simulation reproduces real data properties.
when_to_use_negative:
- The goal is to assess scalability or usability of a simulator rather than fidelity
  — the benchmark treats these as separate evaluation dimensions
- You need a simulation method itself; use simmethods/simpipe to generate data first
  — this skill only evaluates the simulated output against the real reference
- Data are not single-cell/spatial transcriptomics expression data (e.g., bulk RNA-seq
  or non-transcriptomic data), since the 15 data properties are tailored to scRNA-seq/SRT
  characteristics such as zero proportions and library size
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: simpipe
  role: Provides the standard pipeline and the 8 accuracy metrics used to evaluate
    simulated data against real references from general and functional perspectives
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Collection of the 49 simulation methods whose simulated outputs are compared
    against real references
  repo: https://github.com/duohongrui/simmethods
- name: simpipe2docker
  role: Links the R environment to the simpipe Docker container so the evaluation
    pipeline can run reproducibly in Docker
  repo: https://github.com/duohongrui/simpipe2docker
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:243d6cb9be26cd7d0da6b7b65c6fe28c219ae4f134fb993f93c2676c6b9c4805
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# multi-metric-data-property-comparison

## Summary

Evaluates the accuracy of scRNA-seq/SRT simulation methods by comparing each simulated dataset against its real reference dataset across 15 data properties using the 8 accuracy metrics implemented in simpipe, then aggregating metric results into a single accuracy score per method.

## When to use

Use when you have paired simulated and real reference single-cell RNA-seq or spatially resolved transcriptomics datasets and need to quantify how faithfully the simulation reproduces real data properties.

## When NOT to use

- The goal is to assess scalability or usability of a simulator rather than fidelity — the benchmark treats these as separate evaluation dimensions
- You need a simulation method itself; use simmethods/simpipe to generate data first — this skill only evaluates the simulated output against the real reference
- Data are not single-cell/spatial transcriptomics expression data (e.g., bulk RNA-seq or non-transcriptomic data), since the 15 data properties are tailored to scRNA-seq/SRT characteristics such as zero proportions and library size

## Inputs

- Real reference scRNA-seq or SRT datasets (e.g., the 152 reference datasets from 24 platforms, Additional file 2, Tables S1–S3 of DOI 10.1186/s13059-024-03290-y)
- Simulated datasets generated from those references via simmethods/simpipe wrappers

## Outputs

- Per-dataset comparison results across 15 data properties and 8 accuracy metrics
- Aggregated accuracy score per simulation method (overall and stratified by scRNA-seq vs SRT), e.g., written to accuracy_scores.csv

## How to apply

Install the simpipe and simmethods R packages from GitHub with devtools (plus tislingshot if the data carry trajectory information, or use the simpipe Docker image linked via simpipe2docker). For each of the 49 collected simulation methods, run the simpipe wrappers on the applicable reference datasets (the benchmark used 152 real reference datasets from 24 platforms) and collect the simulated output. Compare each simulated dataset against its real reference across 15 data properties using the 8 accuracy metrics implemented in simpipe, covering general and functional perspectives. Aggregate the metric results into a single accuracy score per method, overall and stratified by scRNA-seq vs SRT technique; for calibration, the published benchmark reports SRTsim at 0.84, scDesign3-tree at 0.78, ZINB-WaVE at 0.77, scDesign3 at 0.76, and scDesign2 at 0.74, with Class 2 methods averaging 0.60.

## Related tools

- **simpipe** (Provides the standard pipeline and the 8 accuracy metrics used to evaluate simulated data against real references from general and functional perspectives) — https://github.com/duohongrui/simpipe
- **simmethods** (Collection of the 49 simulation methods whose simulated outputs are compared against real references) — https://github.com/duohongrui/simmethods
- **simpipe2docker** (Links the R environment to the simpipe Docker container so the evaluation pipeline can run reproducibly in Docker) — https://github.com/duohongrui/simpipe2docker

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- The aggregated accuracy ranking reproduces the published top performers: SRTsim 0.84, scDesign3-tree 0.78, ZINB-WaVE 0.77, scDesign3 0.76, scDesign2 0.74, with Class 2 mean 0.60
- Every simulated dataset is compared against its real reference on the full set of 15 data properties using all 8 simpipe accuracy metrics
- Results are aggregated both overall and stratified by scRNA-seq vs SRT technique and exported (e.g., to accuracy_scores.csv)
- Methods known to be unrealistic in the paper (e.g., Phenopath, Lun, Simple, MFA — high scalability but not realistic) do not score at the top of the accuracy ranking
- Simulation runs that fail due to failed parameter estimations or missing/infinite values are identified and handled rather than silently scored

## Limitations

- Execution errors are mainly caused by failed parameter estimations and the appearance of missing or infinite values in calculations
- No single method performs best on all criteria; users must consider trade-offs between accuracy and scalability (or functionality)
- Some methods cannot accurately reproduce key real-data properties (e.g., Simple fails on library size and per-cell zero proportions; scDD cannot characterize high proportions of zeros; BASiCS does not characterize sparsity)

## Evidence

- [readme] The goal of simpipe is to establish a standard pipeline to estimating parameters from real datasets, simulating single-cell RNA-seq datasets and evaluating the simulated data from general and functional perspectives.: "The goal of simpipe is to establish a standard pipeline to estimating parameters from real datasets, simulating single-cell RNA-seq datasets and evaluating the simulated data"
- [abstract] Abstract benchmark scope: "We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability using 152 reference"
- [abstract] Best performers on accuracy: "SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across various platforms"
- [abstract] Scalability-vs-accuracy caveat: "Phenopath, Lun, Simple, and MFA yield high scalability scores but they cannot generate realistic simulated data"
- [abstract] Failure mode: "execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations"
