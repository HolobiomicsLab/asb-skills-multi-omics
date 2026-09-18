---
name: simulated-ground-truth-fidelity-scoring
description: Use when you have simulated scRNA-seq/SRT datasets with known ground-truth
  annotations (groups, DEGs, batches, trajectory) and need to rank simulation methods
  by a functionality score.
when_to_use_negative:
- Evaluating accuracy (fidelity to real-data properties such as library size or zero
  proportion) rather than functionality in the four simulation scenarios — that is
  a different criterion in the benchmark.
- The simulated data lack ground-truth annotations (groups, DEGs, batches, or trajectory),
  so scenario metrics like kBET, ASW-batch, or F1milestones cannot be computed.
- Benchmarking scalability or usability (e.g., runtime and memory of Phenopath, Lun,
  Simple, MFA) instead of functionality scores.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: simpipe
  role: Standard pipeline for estimating parameters from real data, simulating datasets,
    and evaluating simulated data from general and functional perspectives
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Collection of 49 simulation methods used to generate datasets with cell groups,
    batches, DEGs, and differentiation trajectories
  repo: https://github.com/duohongrui/simmethods
- name: tislingshot
  role: Slingshot-based trajectory inference used by simpipe to compute trajectory
    metrics (HIM, F1branches, F1milestones, Cordist); must be installed before simulating
    trajectory data
  repo: https://github.com/dynverse/ti_slingshot/package/
- name: simpipe2docker
  role: Links the R environment to the simpipe Docker container to run the simulation-evaluation
    pipeline reproducibly
  repo: https://github.com/duohongrui/simpipe2docker
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:91f3ece75aae7464171ae8f3d2f8ab0686e624f057cf7c67899dd0a86604a383
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simpipe
  tools:
  - Lun
  - SPARSim
  - simmethods
  - simpipe
  - simpipe2docker
  - tislingshot
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# simulated-ground-truth-fidelity-scoring

## Summary

Aggregates scenario-specific evaluation metrics computed on simulated single-cell/spatial transcriptomics datasets with ground truth into a per-method functionality score for four scenarios (cell groups, DEGs, cell batches, trajectories), as done in the simmethods/simpipe benchmark of 49 simulation methods.

## When to use

Use when you have simulated scRNA-seq/SRT datasets with known ground-truth annotations (groups, DEGs, batches, trajectory) and need to rank simulation methods by a functionality score.

## When NOT to use

- Evaluating accuracy (fidelity to real-data properties such as library size or zero proportion) rather than functionality in the four simulation scenarios — that is a different criterion in the benchmark.
- The simulated data lack ground-truth annotations (groups, DEGs, batches, or trajectory), so scenario metrics like kBET, ASW-batch, or F1milestones cannot be computed.
- Benchmarking scalability or usability (e.g., runtime and memory of Phenopath, Lun, Simple, MFA) instead of functionality scores.

## Inputs

- Simulated scRNA-seq/SRT count datasets with ground-truth cell-group, DEG, batch, or trajectory annotations produced by simmethods/simpipe
- Reference (real) datasets used for parameter estimation (152 reference datasets from 24 platforms in the benchmark)

## Outputs

- Per-scenario, per-method functionality scores aggregated from scenario-specific metrics
- functionality_scores.csv

## How to apply

Generate simulated datasets with ground-truth annotations for each evaluated method (e.g., Lun, SPARSim, scMultiSim, Lun2) using the simmethods/simpipe pipeline. Compute scenario-specific metrics: ASW, Dunn, Connectivity, DB, CDI, and ROUGE for the cell-groups and DEG scenarios; edgeRQLFDetRate-ratio, distribution-score, and SVM for DEGs; CMS, LISI, MM, entropy, kBET, ASW-batch, and PCR for cell batches; and Slingshot-based HIM, F1branches, F1milestones, and Cordist for the trajectory scenario. Aggregate each method's metric results into a per-scenario functionality score and write the results to functionality_scores.csv. Validate against reported benchmarks (Lun: 0.71 for cell groups and 0.90 for DEGs; SPARSim: 0.77 for cell batches; scMultiSim: 0.56; Lun2: 0.53) and confirm that Class1 methods perform poorly on F1milestones and Cordist.

## Related tools

- **simpipe** (Standard pipeline for estimating parameters from real data, simulating datasets, and evaluating simulated data from general and functional perspectives) — https://github.com/duohongrui/simpipe
- **simmethods** (Collection of 49 simulation methods used to generate datasets with cell groups, batches, DEGs, and differentiation trajectories) — https://github.com/duohongrui/simmethods
- **tislingshot** (Slingshot-based trajectory inference used by simpipe to compute trajectory metrics (HIM, F1branches, F1milestones, Cordist); must be installed before simulating trajectory data) — https://github.com/dynverse/ti_slingshot/package/
- **simpipe2docker** (Links the R environment to the simpipe Docker container to run the simulation-evaluation pipeline reproducibly) — https://github.com/duohongrui/simpipe2docker

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe"); devtools::install_github("dynverse/ti_slingshot/package/")
```

## Evaluation signals

- Reproduced functionality scores match reported values: Lun = 0.71 (cell groups) and 0.90 (DEGs); SPARSim = 0.77 (cell batches); scMultiSim = 0.56; Lun2 = 0.53.
- Class1 methods show poor performance on the trajectory metrics F1milestones and Cordist.
- All per-method, per-scenario metric results are aggregated and written to functionality_scores.csv.
- Simulated datasets carry correct ground-truth annotations (cell groups, DEG labels, batch labels, trajectory structure) so scenario metrics are computable.
- Execution errors, when they occur, are attributable to failed parameter estimations or missing/infinite values rather than pipeline misuse.

## Limitations

- Lun does not account for the sparsity and biological variability of scRNA-seq data, despite its top functionality scores.
- Some methods relying on customized parameters (Splat, SCRIP, Lun, scMultiSim) may cause data distortion and over-simulation when generating ground-truth datasets.
- Execution errors are mainly caused by failed parameter estimations and the appearance of missing or infinite values in metric calculations.
- No method performs best on all criteria; trade-offs between accuracy and functionality/scalability must be considered.

## Evidence

- [results] Lun was the best method for simulating datasets with specified cell groups (score = 0.71) and DEGs (score = 0.90).: "Lun was the best method for simulating datasets with specified cell groups (score = 0.71) and DEGs (score = 0.90)."
- [readme] Simpipe pipeline installation instructions: "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [methods] Trajectory prerequisite install: "if you want to use simpipe to simulate datasets with **trajectory information**, please install **tislingshot** first"
- [readme] Simpipe goal statement: "The goal of simpipe is to establish a standard pipeline to estimating parameters from real datasets, simulating single-cell RNA-seq datasets and evaluating the simulated data from general and"
- [abstract] Execution error causes: "execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations"
