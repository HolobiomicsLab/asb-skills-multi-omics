---
name: single-cell-simulation-scenario-benchmarking
description: Use when you need to rank or compare scRNA-seq/SRT simulation methods
  by functionality — e.g., which simulator highest-ranked reproduces cell groups,
  DEGs, batches, or differentiation trajectories — and want reproducible scores for
  each method per scenario.
when_to_use_negative:
- The goal is accuracy (distribution fit to real data) or scalability/usability benchmarking
  rather than functionality under the four scenarios — the paper evaluates these as
  separate dimensions.
- You need to simulate spatial transcriptomics data with spatial coordinates where
  the four scRNA-seq functionality scenarios do not apply.
- A single simulator already fits the need and no cross-method comparison is required.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: simmethods
  role: R package collecting the 49 simulation methods used to generate simulated
    datasets with ground truth in each scenario
  repo: https://github.com/duohongrui/simmethods
- name: simpipe
  role: Standard pipeline for estimating parameters from real data, simulating, and
    evaluating simulated data from general and functional perspectives
  repo: https://github.com/duohongrui/simpipe
- name: tislingshot
  role: Required dependency for using simpipe to simulate datasets with trajectory
    information
  repo: https://github.com/dynverse/ti_slingshot/package/
- name: simpipe2docker
  role: Links the R environment to the simpipe Docker container so simulations run
    reproducibly in Docker
  repo: https://github.com/duohongrui/simpipe2docker
- name: Lun
  role: Top-performing method in the cell-groups (score 0.71) and DEGs (score 0.90)
    functionality scenarios
- name: SPARSim
  role: Top-performing method in the cell-batches functionality scenario (score 0.77)
- name: scMultiSim
  role: Evaluated method whose functionality score (0.56) serves as a reproduction
    target
- name: Lun2
  role: Evaluated method whose functionality score (0.53) serves as a reproduction
    target
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:0f8fb358e6cab38a7f9dbf83f1fa30bac32420cba9077eef52b2555bc4600f4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simmethods
  tools:
  - Lun
  - Lun2
  - SPARSim
  - scMultiSim
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-simulation-scenario-benchmarking

## Summary

Benchmark the functionality of single-cell RNA-seq and spatial transcriptomics simulation methods across four scenarios (cell groups, DEGs, cell batches, trajectories) using the simmethods/simpipe stack, aggregating scenario-specific metrics into per-method functionality scores.

## When to use

Use when you need to rank or compare scRNA-seq/SRT simulation methods by functionality — e.g., which simulator best reproduces cell groups, DEGs, batches, or differentiation trajectories — and want reproducible scores for each method per scenario.

## When NOT to use

- The goal is accuracy (distribution fit to real data) or scalability/usability benchmarking rather than functionality under the four scenarios — the paper evaluates these as separate dimensions.
- You need to simulate spatial transcriptomics data with spatial coordinates where the four scRNA-seq functionality scenarios do not apply.
- A single simulator already fits the need and no cross-method comparison is required.

## Inputs

- Real reference scRNA-seq or SRT datasets (from the paper's 152 reference datasets across 24 platforms) for parameter estimation
- List of simulation methods to evaluate (e.g., Lun, SPARSim, scMultiSim, Lun2) from the 49 collected in simmethods
- Scenario specification: cell-groups, DEGs, cell-batches, or trajectory

## Outputs

- functionality_scores.csv with per-method, per-scenario functionality scores
- Per-scenario metric values (ASW, Dunn, Connectivity, DB, CDI, ROUGE, edgeRQLFDetRate-ratio, distribution-score, SVM, CMS, LISI, MM, entropy, kBET, ASW-batch, PCR, HIM, F1branches, F1milestones, Cordist)
- Simulated datasets with ground-truth annotations per scenario

## How to apply

Install the stack with devtools::install_github for duohongrui/simmethods and duohongrui/simpipe, plus tislingshot (dynverse/ti_slingshot) when the trajectory scenario is included; optionally run through the simpipe Docker image linked via simpipe2docker and consult the Simsite documentation. For each of the four functionality scenarios (cell-groups, DEGs, cell-batches, trajectory), generate simulated datasets with ground-truth annotations using the simmethods/simpipe pipeline for the evaluated methods (e.g., Lun, SPARSim, scMultiSim, Lun2). Compute scenario-specific metrics: ASW, Dunn, Connectivity, DB, CDI, and ROUGE for cell-groups and DEG scenarios; edgeRQLFDetRate-ratio, distribution-score, and SVM for DEGs; CMS, LISI, MM, entropy, kBET, ASW-batch, and PCR for cell-batches; and Slingshot-based HIM, F1branches, F1milestones, and Cordist for the trajectory scenario. Aggregate each method's metric results into a per-scenario functionality score and write all scores to functionality_scores.csv, validating against published values (Lun: 0.71 cell-groups, 0.90 DEGs; SPARSim: 0.77 cell-batches; scMultiSim: 0.56; Lun2: 0.53).

## Related tools

- **simmethods** (R package collecting the 49 simulation methods used to generate simulated datasets with ground truth in each scenario) — https://github.com/duohongrui/simmethods
- **simpipe** (Standard pipeline for estimating parameters from real data, simulating, and evaluating simulated data from general and functional perspectives) — https://github.com/duohongrui/simpipe
- **tislingshot** (Required dependency for using simpipe to simulate datasets with trajectory information) — https://github.com/dynverse/ti_slingshot/package/
- **simpipe2docker** (Links the R environment to the simpipe Docker container so simulations run reproducibly in Docker) — https://github.com/duohongrui/simpipe2docker
- **Lun** (Top-performing method in the cell-groups (score 0.71) and DEGs (score 0.90) functionality scenarios)
- **SPARSim** (Top-performing method in the cell-batches functionality scenario (score 0.77))
- **scMultiSim** (Evaluated method whose functionality score (0.56) serves as a reproduction target)
- **Lun2** (Evaluated method whose functionality score (0.53) serves as a reproduction target)

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe"); devtools::install_github("dynverse/ti_slingshot/package/")
```

## Evaluation signals

- Reproduced scores match the reported values: Lun 0.71 (cell-groups) and 0.90 (DEGs); SPARSim 0.77 (cell-batches); scMultiSim 0.56; Lun2 0.53.
- Top-ranked method per scenario is Lun for cell-groups and DEGs, SPARSim for cell-batches.
- Class1 methods show poor performance on F1milestones and Cordist in the trajectory scenario.
- All metric values are finite (no missing or infinite values in calculations, the main cause of execution errors).
- functionality_scores.csv is written with one aggregated score per method per scenario.

## Limitations

- Execution errors are mainly caused by failed parameter estimations and the appearance of missing or infinite values in calculations.
- No method performs best on all criteria, so a good-yet-not-the-best method should be chosen when it solves the problem effectively; accuracy and scalability/functionality trade-offs must be considered.
- Methods relying solely on customized parameters (Splat, SCRIP, Lun, scMultiSim) may cause data distortion and over-simulation when generating ground-truth datasets; excessively flexible parameter settings can distort the data so it no longer reflects real biological changes.
- Trajectory simulation requires the extra tislingshot dependency, installed from dynverse/ti_slingshot.

## Evidence

- [abstract] 49 simulation methods evaluated on 152 reference datasets: "We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability using 152 reference datasets derived from 24"
- [abstract] Lun, SPARSim, and scDesign3-tree outperform other methods: "Lun, SPARSim, and scDesign3-tree outperform other methods under corresponding simulation scenarios"
- [methods] Install simmethods and simpipe: "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [readme] Trajectory simulation requires tislingshot: "if you want to use simpipe to simulate datasets with **trajectory information**, please install **tislingshot** first"
- [readme] simpipe goal: "The goal of simpipe is to establish a standard pipeline to estimating parameters from real datasets, simulating single-cell RNA-seq datasets and evaluating the simulated data from general and"
