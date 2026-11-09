---
name: trajectory-aware-dataset-simulation
description: Use when the analysis goal is generating synthetic scRNA-seq datasets
  with differentiation-trajectory ground truth (e.g., for benchmarking trajectory
  inference), starting from a real reference dataset whose parameters can be estimated.
when_to_use_negative:
- The target simulation only needs group, batch, or differential-expression structure
  without differentiation trajectory ground truth (plain simpipe/simmethods usage
  suffices and tislingshot is unnecessary)
- No real reference dataset is available for parameter estimation, so the simulation
  would rely solely on customized parameters, which the paper warns can cause data
  distortion and over-simulation
- The goal is simulating spatially resolved transcriptomics (SRT) data, where top
  performers were SRTsim, scDesign3, ZINB-WaVE, and scDesign2 rather than the trajectory-focused
  simulators
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: simpipe
  role: Standard pipeline for estimating parameters from real data, simulating datasets,
    and evaluating simulated data from general and functional perspectives; requires
    tislingshot for trajectory-information scenarios
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Collection of 49 simulation methods for single-cell and spatial transcriptome
    data, including trajectory-capable simulators (dyngen, dyntoy, ESCO-traj, ESCO-tree),
    invoked through simpipe
  repo: https://github.com/duohongrui/simmethods
- name: tislingshot
  role: Mandatory prerequisite for using simpipe to simulate datasets with trajectory
    information
  repo: https://github.com/dynverse/ti_slingshot/package/
- name: simpipe2docker
  role: 'Alternative Docker-based route: links the R environment to the simpipe Docker
    container so trajectory simulations run in a prebuilt environment'
  repo: https://github.com/duohongrui/simpipe2docker
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:e0c8bddf02716f96f241cd08b9631a1b94a5d935651c1adbadcfd5abafc12d14
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
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# trajectory-aware-dataset-simulation

## Summary

Simulate single-cell RNA-seq datasets that carry differentiation-trajectory ground truth (pseudotime and lineage topology) by estimating parameters from a real reference dataset and running trajectory-capable simulators wrapped in the simpipe/simmethods pipeline.

## When to use

Use when the analysis goal is generating synthetic scRNA-seq datasets with differentiation-trajectory ground truth (e.g., for benchmarking trajectory inference), starting from a real reference dataset whose parameters can be estimated.

## When NOT to use

- The target simulation only needs group, batch, or differential-expression structure without differentiation trajectory ground truth (plain simpipe/simmethods usage suffices and tislingshot is unnecessary)
- No real reference dataset is available for parameter estimation, so the simulation would rely solely on customized parameters, which the paper warns can cause data distortion and over-simulation
- The goal is simulating spatially resolved transcriptomics (SRT) data, where top performers were SRTsim, scDesign3, ZINB-WaVE, and scDesign2 rather than the trajectory-focused simulators

## Inputs

- Real reference scRNA-seq dataset with a differentiation trajectory (used for parameter estimation via simpipe)
- Choice of trajectory-capable simulation method from the simmethods collection (e.g., dyngen, dyntoy, ESCO-traj)
- Simulation parameters (number of cells, genes, trajectory topology/ground truth settings)

## Outputs

- Simulated scRNA-seq dataset with trajectory ground truth (pseudotime/lineage labels)
- Accuracy evaluation results comparing simulated data against the real reference over 15 data properties with 8 simpipe metrics

## How to apply

Install the simmethods and simpipe R packages with devtools, and before running any trajectory-informed simulation, install tislingshot (from dynverse/ti_slingshot) since simpipe requires it specifically for datasets with trajectory information; alternatively pull the simpipe Docker image and link R via simpipe2docker. Simmethods bundles trajectory-capable simulators (e.g., dyngen, dyntoy, ESCO-traj, ESCO-tree variants) among its 49 collected methods. Estimate parameters from a real scRNA-seq reference dataset with simpipe, then simulate new data containing differentiation trajectory ground truth. Evaluate fidelity by comparing simulated data to the real reference across 15 data properties using the 8 accuracy metrics implemented in simpipe, and prefer good-yet-not-the-best methods since no single simulator wins on all criteria. Be aware that methods relying solely on customized parameters or kinetic models (topology and kinetic parameters) may distort data or neglect gene-expression and variability characteristics of the real data.

## Related tools

- **simpipe** (Standard pipeline for estimating parameters from real data, simulating datasets, and evaluating simulated data from general and functional perspectives; requires tislingshot for trajectory-information scenarios) — https://github.com/duohongrui/simpipe
- **simmethods** (Collection of 49 simulation methods for single-cell and spatial transcriptome data, including trajectory-capable simulators (dyngen, dyntoy, ESCO-traj, ESCO-tree), invoked through simpipe) — https://github.com/duohongrui/simmethods
- **tislingshot** (Mandatory prerequisite for using simpipe to simulate datasets with trajectory information) — https://github.com/dynverse/ti_slingshot/package/
- **simpipe2docker** (Alternative Docker-based route: links the R environment to the simpipe Docker container so trajectory simulations run in a prebuilt environment) — https://github.com/duohongrui/simpipe2docker

## Examples

```
devtools::install_github("dynverse/ti_slingshot/package/")
```

## Evaluation signals

- tislingshot is installed before any trajectory simulation run (simpipe raises the requirement explicitly)
- The simulated dataset carries trajectory ground truth (differentiation trajectory/pseudotime labels) alongside the expression matrix
- Simulated data reproduce real-data properties (e.g., library size, per-cell zero proportions) when assessed with simpipe's 15 data properties and 8 accuracy metrics
- Execution completes without the common failure modes: failed parameter estimations or missing/infinite values appearing in calculations
- For tree/lineage-based scenarios, accuracy is consistent with the paper's finding that scDesign3-tree outperforms other methods under corresponding simulation scenarios

## Limitations

- Kinetic-model-based trajectory simulators simulate from the provided topology structure and kinetic parameters, neglecting characterization of gene expression and variability from the real data
- Methods relying solely on customized parameters (Splat, SCRIP, Lun, scMultiSim) may cause data distortion and over-simulation when generating ground-truth datasets
- Execution errors are mainly caused by failed parameter estimations and the appearance of missing or infinite values in calculations
- No method performs best on all criteria, so trajectory simulators must be chosen with trade-offs between accuracy, scalability, and functionality in mind

## Evidence

- [methods] tislingshot prerequisite for trajectory simulation: "if you want to use simpipe to simulate datasets with **trajectory information**, please install **tislingshot** first"
- [readme] simpipe pipeline scope: "The goal of simpipe is to establish a standard pipeline to estimating parameters from real datasets, simulating single-cell RNA-seq datasets and evaluating the simulated data"
- [readme] trajectory as a supported simulation scenario: [evidence span withheld — anchor status: external_doc]
- [discussion] distortion risk with customized-parameter methods: "Some simulation algorithms may cause data distortion and over-simulation when generating datasets with ground truth."
