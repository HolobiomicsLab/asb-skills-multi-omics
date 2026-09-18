---
name: spatial-transcriptomics-data-simulation
description: Use when you need to generate synthetic SRT or scRNA-seq datasets that
  preserve real-data properties (e.g. from a reference dataset), or when choosing
  which of 49 simulation methods fits your scenario (groups, batches, trajectory,
  SRT).
when_to_use_negative:
- When a small, fast simulation matters more than fidelity — Phenopath, Lun, Simple,
  and MFA yield high scalability scores but cannot generate realistic simulated data
- When the target data property is sparsity and you are restricted to BASiCS or scDD
  — BASiCS does not characterize zero-value properties, and scDD only samples genes
  with <75% zeros
- When you rely on purely customized parameters without reference-data estimation
  (e.g. Splat, SCRIP, Lun, scMultiSim), which can cause data distortion and over-simulation
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: simmethods
  role: R package collecting and wrapping the 49 simulation methods; install via devtools
    from GitHub
  repo: https://github.com/duohongrui/simmethods
- name: simpipe
  role: Standard pipeline for estimating parameters from real data, simulating datasets,
    and evaluating simulated data from general and functional perspectives
  repo: https://github.com/duohongrui/simpipe
- name: SRTsim
  role: Top-accuracy simulator for SRT data (accuracy score 0.84 in the benchmark)
- name: scDesign3
  role: High-accuracy simulator (0.76; scDesign3-tree variant 0.78) applicable across
    platforms
- name: ZINB-WaVE
  role: High-accuracy simulator (0.77) with low execution-error rate (0.7%)
- name: scDesign2
  role: High-accuracy simulator (0.74) across various platforms
- name: tislingshot
  role: Trajectory-information simulation backend; must be installed before simulating
    datasets with trajectory information
  repo: https://github.com/dynverse/ti_slingshot/package/
- name: simpipe2docker
  role: Links the R environment to the simpipe Docker container for a consistent execution
    environment
  repo: https://github.com/duohongrui/simpipe2docker
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:d95ecce5546ac3ccb77d39ebf5e02b2954d466b1a586caf0775a5127a1e82a04
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
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# spatial-transcriptomics-data-simulation

## Summary

Simulate spatially resolved transcriptomics (SRT) data by estimating parameters from real reference datasets and running benchmarked simulators (e.g. SRTsim, scDesign3, ZINB-WaVE, scDesign2) through the simmethods/simpipe R pipeline, then evaluating fidelity of the simulated data.

## When to use

Use when you need to generate synthetic SRT or scRNA-seq datasets that preserve real-data properties (e.g. from a reference dataset), or when choosing which of 49 simulation methods fits your scenario (groups, batches, trajectory, SRT).

## When NOT to use

- When a small, fast simulation matters more than fidelity — Phenopath, Lun, Simple, and MFA yield high scalability scores but cannot generate realistic simulated data
- When the target data property is sparsity and you are restricted to BASiCS or scDD — BASiCS does not characterize zero-value properties, and scDD only samples genes with <75% zeros
- When you rely on purely customized parameters without reference-data estimation (e.g. Splat, SCRIP, Lun, scMultiSim), which can cause data distortion and over-simulation

## Inputs

- Real reference scRNA-seq or SRT dataset (count matrix, e.g. from the 152 reference datasets of the benchmark)
- Simulation scenario specification: number of groups, batches, differential expressed genes, cell-type tree, or differentiation trajectory
- Choice of simulation method from the 49 collected in simmethods (e.g. SRTsim, scDesign3, ZINB-WaVE, scDesign2, Lun, SPARSim)

## Outputs

- Simulated single-cell or spatial transcriptomics count dataset with ground-truth labels (cell groups, batches, trajectory pseudotime, or spatial coordinates)
- Estimated parameters from the real reference data
- Evaluation results comparing simulated vs real data across 15 data properties using 8 accuracy metrics

## How to apply

Install simmethods and simpipe from GitHub with devtools (devtools::install_github('duohongrui/simmethods'); devtools::install_github('duohongrui/simpipe')); for trajectory-informed simulation also install tislingshot (dynverse/ti_slingshot/package/). Provide a real reference scRNA-seq or SRT dataset (the benchmark used 152 reference datasets from 24 platforms) and let simpipe estimate parameters from it, simulate new data with the chosen method, and evaluate the result. Prefer SRTsim, scDesign3, ZINB-WaVE, or scDesign2 for accuracy (SRTsim scored highest, 0.84, followed by scDesign3-tree 0.78, ZINB-WaVE 0.77, scDesign3 0.76, scDesign2 0.74); use Lun, SPARSim, or scDesign3-tree for scenario-specific performance. For a reproducible environment, pull the duohongrui/simpipe Docker image and link it to R via simpipe2docker. Judge success by comparing simulated data against the real reference across 15 data properties with simpipe's 8 accuracy metrics, and check that methods like Phenopath, Simple, and MFA — despite high scalability — are avoided when realistic data is required.

## Related tools

- **simmethods** (R package collecting and wrapping the 49 simulation methods; install via devtools from GitHub) — https://github.com/duohongrui/simmethods
- **simpipe** (Standard pipeline for estimating parameters from real data, simulating datasets, and evaluating simulated data from general and functional perspectives) — https://github.com/duohongrui/simpipe
- **SRTsim** (Top-accuracy simulator for SRT data (accuracy score 0.84 in the benchmark))
- **scDesign3** (High-accuracy simulator (0.76; scDesign3-tree variant 0.78) applicable across platforms)
- **ZINB-WaVE** (High-accuracy simulator (0.77) with low execution-error rate (0.7%))
- **scDesign2** (High-accuracy simulator (0.74) across various platforms)
- **tislingshot** (Trajectory-information simulation backend; must be installed before simulating datasets with trajectory information) — https://github.com/dynverse/ti_slingshot/package/
- **simpipe2docker** (Links the R environment to the simpipe Docker container for a consistent execution environment) — https://github.com/duohongrui/simpipe2docker

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- Overall accuracy score per method should reproduce the reported ranking: SRTsim 0.84, scDesign3-tree 0.78, ZINB-WaVE 0.77, scDesign3 0.76, scDesign2 0.74 (Class 2 mean 0.60)
- Simulated data should match the real reference across the 15 data properties using simpipe's 8 accuracy metrics
- Simulated ground truth (group/batch/trajectory/spatial labels) must be consistent with the scenario requested
- Execution-error rate check: about 61% of methods encountered errors; a wrapper that captures error messages verbatim should classify failures into the 10 defined error categories (dominated by missing/infinite values and failed gene parameter estimation)
- Docker route check: simpipe2docker::test_docker_installation(detailed = TRUE) must report all checks TRUE before running containerized simulations

## Limitations

- No consensus on distributional assumptions for modeling gene expression (NB vs Poisson; zero-inflation controversy for plate-based data); a single parametric distribution may lose information about data properties
- Excessively flexible parameter settings can distort simulated data so it no longer reflects real biological changes or differences
- Kinetic-model-based methods simulate solely from provided topology and kinetic parameters, neglecting characterization of gene expression and variability from real data
- No method performs best on all criteria; users should consider trade-offs between accuracy and scalability (or functionality)

## Evidence

- [abstract] SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across various platforms: "SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across various platforms"
- [abstract] Lun, SPARSim, and scDesign3-tree outperform other methods under corresponding simulation scenarios: "Lun, SPARSim, and scDesign3-tree outperform other methods under corresponding simulation scenarios"
- [abstract] Phenopath, Lun, Simple, and MFA yield high scalability scores but they cannot generate realistic simulated data: "Phenopath, Lun, Simple, and MFA yield high scalability scores but they cannot generate realistic simulated data"
- [readme] devtools install commands from README: "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [readme] tislingshot installation instruction: "if you want to use simpipe to simulate datasets with **trajectory information**, please install **tislingshot** first"
- [readme] simpipe2docker purpose: [evidence span withheld — anchor status: external_doc]
- [readme] simmethods scope: [evidence span withheld — anchor status: external_doc]
- [methods] install commands from methods: "devtools::install_github("duohongrui/simpipe")"
