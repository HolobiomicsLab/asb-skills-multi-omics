---
name: single-cell-rna-seq-data-simulation
description: Use when a downstream method (clustering, trajectory inference, DE analysis)
  needs synthetic scRNA-seq/SRT data with ground truth for groups, batches, DE genes,
  or differentiation trajectories, and method choice must be justified by benchmark
  evidence.
when_to_use_negative:
- The goal is to benchmark simulators themselves rather than generate data — use the
  full simpipe accuracy/functionality/scalability/usability benchmark protocol instead
- 'The input is not real scRNA-seq/SRT expression data and no reference dataset is
  available for parameter estimation (e.g. purely kinetic-parameter-driven simulation
  is discouraged: such methods neglect characterization of gene expression and variability
  from real data)'
- Only customized-parameter simulation (e.g. Splat, SCRIP, Lun, scMultiSim defaults)
  is acceptable when faithful reproduction of real-data properties is required, since
  this can cause data distortion and over-simulation
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_2229
tools:
- name: simmethods
  role: R package collecting and wrapping the 49 simulation methods for single-cell
    and spatial transcriptome data, including group, batch, DE-gene, and trajectory
    scenarios
  repo: https://github.com/duohongrui/simmethods
- name: simpipe
  role: Standard pipeline for estimating parameters from real data, simulating datasets,
    and evaluating simulated data from general and functional perspectives
  repo: https://github.com/duohongrui/simpipe
- name: simpipe2docker
  role: Links the R environment to the simpipe Docker container for a consistent execution
    environment across all 49 methods
  repo: https://github.com/duohongrui/simpipe2docker
- name: tislingshot
  role: Installs Slingshot trajectory inference so simpipe can simulate datasets with
    trajectory information
  repo: https://github.com/duohongrui/simpipe
- name: SRTsim
  role: Top-accuracy simulator for SRT data (accuracy score 0.84) across platforms
- name: scDesign3
  role: High-accuracy scRNA-seq simulator (0.76; scDesign3-tree variant 0.78 under
    corresponding scenarios)
- name: ZINB-WaVE
  role: High-accuracy simulator (0.77) with very low execution-error rate (0.7%)
- name: scDesign2
  role: High-accuracy simulator (0.74) across platforms
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:12c80793a8aa587ed7155b64be198acd63ee67330161b61fd494242dd1a7566a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simmethods
  tools:
  - Docker
  - Lun
  - MFA
  - Phenopath
  - R
  - SPARSim
  - SRTsim
  - Simple
  - ZINB-WaVE
  - devtools
  - scDesign2
  - scDesign3
  - scDesign3-tree
  - simmethods
  - simpipe
  - simpipe2docker
  - tislingshot
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# single-cell-rna-seq-data-simulation

## Summary

Simulate single-cell RNA-seq (and spatially resolved transcriptomics) datasets with realistic ground truth using the simmethods/simpipe ecosystem of 49 benchmarked simulators, selecting methods based on accuracy, functionality, scalability, and usability trade-offs established in the Genome Biology 2024 benchmark of 49 methods on 152 reference datasets from 24 platforms.

## When to use

Use when a downstream method (clustering, trajectory inference, DE analysis) needs synthetic scRNA-seq/SRT data with ground truth for groups, batches, DE genes, or differentiation trajectories, and method choice must be justified by benchmark evidence.

## When NOT to use

- The goal is to benchmark simulators themselves rather than generate data — use the full simpipe accuracy/functionality/scalability/usability benchmark protocol instead
- The input is not real scRNA-seq/SRT expression data and no reference dataset is available for parameter estimation (e.g. purely kinetic-parameter-driven simulation is discouraged: such methods neglect characterization of gene expression and variability from real data)
- Only customized-parameter simulation (e.g. Splat, SCRIP, Lun, scMultiSim defaults) is acceptable when faithful reproduction of real-data properties is required, since this can cause data distortion and over-simulation

## Inputs

- Real scRNA-seq or SRT reference count matrix (one of the 152 reference datasets from Additional file 2, Tables S1–S3) used for parameter estimation
- Simulation scenario specification: number of groups, batches, differential expressed genes, or differentiation trajectory information
- Optional cell-level metadata (batch labels, group labels, pseudotime) for group/batch/trajectory-informed simulation

## Outputs

- Simulated count matrix with ground-truth cell/group/trajectory annotations
- Parameter estimates fitted from the reference dataset
- Accuracy evaluation of the simulated data across 15 data properties (e.g. library size, per-cell zero proportions) via simpipe's 8 metrics
- Log of captured execution error messages for per-method error-rate tracking

## How to apply

Install the R packages with devtools::install_github('duohongrui/simmethods') and devtools::install_github('duohongrui/simpipe'), plus dynverse/ti_slingshot if trajectory-informed simulation is needed. Estimate parameters from a real reference dataset (the benchmark used 152 scRNA-seq and SRT reference datasets from 24 platforms), run the chosen simulator via simpipe wrappers (49 methods, e.g. SRTsim, scDesign3, ZINB-WaVE, scDesign2, Lun, SPARSim, scDesign3-tree), and evaluate the simulated data across 15 data properties with the 8 accuracy metrics implemented in simpipe. For reproducible environments, pull the duohongrui/simpipe Docker image and link it to R with simpipe2docker. Prefer SRTsim (accuracy 0.84), scDesign3-tree (0.78), ZINB-WaVE (0.77), scDesign3 (0.76), or scDesign2 (0.74) for accuracy; Lun, SPARSim, and scDesign3-tree outperform under corresponding simulation scenarios; use Phenopath, Simple, or MFA only when scalability outweighs fidelity. Because no method performs best on all criteria, weigh accuracy against scalability/functionality, and monitor execution errors — about 61% of methods errored (0.7% for SCRIP-BGP-trendedBCV and ZINB-WaVE up to 88.9% for BASiCS), mainly from failed parameter estimation for genes and appearance of missing/infinite values.

## Related tools

- **simmethods** (R package collecting and wrapping the 49 simulation methods for single-cell and spatial transcriptome data, including group, batch, DE-gene, and trajectory scenarios) — https://github.com/duohongrui/simmethods
- **simpipe** (Standard pipeline for estimating parameters from real data, simulating datasets, and evaluating simulated data from general and functional perspectives) — https://github.com/duohongrui/simpipe
- **simpipe2docker** (Links the R environment to the simpipe Docker container for a consistent execution environment across all 49 methods) — https://github.com/duohongrui/simpipe2docker
- **tislingshot** (Installs Slingshot trajectory inference so simpipe can simulate datasets with trajectory information) — https://github.com/duohongrui/simpipe
- **SRTsim** (Top-accuracy simulator for SRT data (accuracy score 0.84) across platforms)
- **scDesign3** (High-accuracy scRNA-seq simulator (0.76; scDesign3-tree variant 0.78 under corresponding scenarios))
- **ZINB-WaVE** (High-accuracy simulator (0.77) with very low execution-error rate (0.7%))
- **scDesign2** (High-accuracy simulator (0.74) across platforms)

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- Simulated data reproduces the reference across 15 data properties measured with simpipe's 8 accuracy metrics, aggregating to an accuracy score comparable to reported values (SRTsim 0.84, scDesign3-tree 0.78, ZINB-WaVE 0.77, scDesign3 0.76, scDesign2 0.74; Class 2 mean 0.60)
- Ground-truth annotations (groups, batches, DE genes, pseudotime) in the simulated data match the scenario parameters requested
- Per-method execution-error rate is recorded; rates should fall within the benchmark range 0.7%–88.9%, with errors classified into the 10 categories of Additional file 2 Table S5 (e.g. failed parameter estimation for genes, appearance of missing/infinite values)
- Library size of cells and per-cell zero proportions are reproduced rather than distorted, since methods like Simple fail on these properties
- Docker-based runs via simpipe2docker pass simpipe2docker::test_docker_installation(detailed = TRUE) before simulation

## Limitations

- About 61% of the 49 simulation methods encountered execution errors, ranging from 0.7% (SCRIP-BGP-trendedBCV, ZINB-WaVE) to 88.9% (BASiCS), mainly caused by failed parameter estimation for genes and appearance of missing/infinite values
- There is no consensus on the distributional assumption for modeling gene expression (NB vs Poisson; zero-inflation controversy for plate-based data), and a single parametric distribution may lose important data-property information
- No method performs best on all criteria: Phenopath, Lun, Simple, and MFA yield high scalability scores but cannot generate realistic simulated data; scDD cannot characterize high zero proportions (samples only genes with <75% zeros), and BASiCS does not characterize sparsity
- Excessively flexible parameter settings or reliance on customized parameters (Splat, SCRIP, Lun, scMultiSim) can distort data and cause over-simulation that fails to reflect real biological changes

## Evidence

- [abstract] 49 simulation methods evaluated using 152 reference datasets from 24 platforms on accuracy, functionality, scalability, and usability: "We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability using 152 reference datasets derived from 24"
- [abstract] Best-accuracy methods finding: "SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across various platforms"
- [methods] Installation commands: "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [methods] Trajectory simulation prerequisite: "if you want to use simpipe to simulate datasets with **trajectory information**, please install **tislingshot** first"
- [readme] simpipe purpose: "The goal of simpipe is to establish a standard pipeline to estimating
parameters from real datasets, simulating single-cell RNA-seq datasets
and evaluating the simulated data from general and"
- [readme] simmethods scope: "Simmethods collects and documents 49 popular and common simulation
methods for single-cell and spatial transcriptome data."
