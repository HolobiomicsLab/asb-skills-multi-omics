---
name: batch-effect-metric-evaluation
description: Use when benchmarking scRNA-seq simulators on their ability to generate
  data with cell batches, i.e. the cell-batches functionality scenario, and you need
  to compute and aggregate batch-preservation metrics into a functionality score.
when_to_use_negative:
- The scenario of interest is simulating specified cell groups or DEGs rather than
  cell batches (use the cell-groups/DEG metrics ASW, Dunn, Connectivity, DB, CDI,
  ROUGE, edgeRQLFDetRate-ratio, distribution-score, SVM instead)
- Evaluating trajectory simulation (use Slingshot-based HIM, F1branches, F1milestones,
  Cordist, which requires installing tislingshot)
- Only assessing accuracy (how well simulated data match real-data distributions)
  rather than functionality; accuracy benchmarking is a separate evaluation dimension
  in this study
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: simmethods
  role: R package collecting the 49 simulation methods; provides the simulators used
    to generate batched scRNA-seq datasets
  repo: https://github.com/duohongrui/simmethods
- name: simpipe
  role: Standard pipeline for estimating parameters from real data, simulating, and
    evaluating simulated data from general and functional perspectives
  repo: https://github.com/duohongrui/simpipe
- name: simpipe2docker
  role: Links the R environment to the simpipe Docker container so simulations and
    evaluations run in a reproducible container
  repo: https://github.com/duohongrui/simpipe2docker
- name: SPARSim
  role: Best-performing method in the cell-batches functionality scenario (score =
    0.77)
- name: Lun
  role: Evaluated simulator; best for cell-groups (0.71) and DEGs (0.90) scenarios
    but not cell batches
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:39abdda357939d754e9409bfa8e009e97f5881a2dab8ddfdc55034c8e01820ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simmethods
  tools:
  - Lun
  - SPARSim
  - simmethods
  - simpipe
  - simpipe2docker
  - tislingshot
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# batch-effect-metric-evaluation

## Summary

Evaluates how well single-cell RNA-seq simulation methods reproduce batch effects, using the simpipe/simmethods stack to generate simulated datasets with cell batches and score them with batch-aware metrics (CMS, LISI, MM, entropy, kBET, ASW-batch, PCR).

## When to use

Use when benchmarking scRNA-seq simulators on their ability to generate data with cell batches, i.e. the cell-batches functionality scenario, and you need to compute and aggregate batch-preservation metrics into a functionality score.

## When NOT to use

- The scenario of interest is simulating specified cell groups or DEGs rather than cell batches (use the cell-groups/DEG metrics ASW, Dunn, Connectivity, DB, CDI, ROUGE, edgeRQLFDetRate-ratio, distribution-score, SVM instead)
- Evaluating trajectory simulation (use Slingshot-based HIM, F1branches, F1milestones, Cordist, which requires installing tislingshot)
- Only assessing accuracy (how well simulated data match real-data distributions) rather than functionality; accuracy benchmarking is a separate evaluation dimension in this study

## Inputs

- Real scRNA-seq reference dataset (with batch information) for parameter estimation
- simmethods/simpipe simulation methods (e.g., Lun, SPARSim, scMultiSim, Lun2)
- Ground-truth batch annotations of the simulated cells

## Outputs

- Simulated scRNA-seq datasets containing cell batches with ground-truth annotations
- Per-metric batch-effect evaluation results (CMS, LISI, MM, entropy, kBET, ASW-batch, PCR)
- Aggregated per-method functionality scores written to functionality_scores.csv

## How to apply

Install the evaluation stack with devtools::install_github("duohongrui/simmethods") and devtools::install_github("duohongrui/simpipe"); optionally run through the simpipe Docker image linked into R via simpipe2docker, consulting the Simsite documentation for per-method usage. Estimate parameters from a real reference dataset, then use simpipe to simulate datasets with cell batches and ground-truth annotations for each evaluated method (e.g., Lun, SPARSim, scMultiSim, Lun2). Compute batch-specific metrics on the simulated data: CMS, LISI, MM, entropy, kBET, ASW-batch, and PCR. Aggregate the metric results into a per-method functionality score and write scores to functionality_scores.csv. In this benchmark, SPARSim achieved the best cell-batches functionality score (0.77).

## Related tools

- **simmethods** (R package collecting the 49 simulation methods; provides the simulators used to generate batched scRNA-seq datasets) — https://github.com/duohongrui/simmethods
- **simpipe** (Standard pipeline for estimating parameters from real data, simulating, and evaluating simulated data from general and functional perspectives) — https://github.com/duohongrui/simpipe
- **simpipe2docker** (Links the R environment to the simpipe Docker container so simulations and evaluations run in a reproducible container) — https://github.com/duohongrui/simpipe2docker
- **SPARSim** (Best-performing method in the cell-batches functionality scenario (score = 0.77))
- **Lun** (Evaluated simulator; best for cell-groups (0.71) and DEGs (0.90) scenarios but not cell batches)

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- Reproduced cell-batches functionality scores must match reported values (SPARSim: 0.77 cell-batches; scMultiSim: 0.56; Lun2: 0.53)
- Lun reproduces its reported functionality scores in other scenarios (0.71 cell-groups, 0.90 DEGs), confirming the shared pipeline is wired correctly
- All seven batch metrics (CMS, LISI, MM, entropy, kBET, ASW-batch, PCR) are computed without missing or infinite values; execution errors in this study are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations
- Per-method scores are aggregated into a single per-scenario functionality score and written to functionality_scores.csv
- Docker validation via simpipe2docker::test_docker_installation(detailed = TRUE) returns TRUE before container-based runs

## Limitations

- Some simulation algorithms may cause data distortion and over-simulation when generating datasets with ground truth, e.g. methods relying solely on customized parameters (Splat, SCRIP, Lun, scMultiSim)
- Confounding technical and biological effects can prevent the simulation method from accurately capturing heterogeneity, affecting reliability and fidelity of simulated data
- Lun does not account for the sparsity and biological variability of scRNA-seq data, which limits interpretation of its scores
- Execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations

## Evidence

- [readme] The goal of simpipe is to establish a standard pipeline to estimating parameters from real datasets: "The goal of simpipe is to establish a standard pipeline to estimating parameters from real datasets, simulating single-cell RNA-seq datasets and evaluating the simulated data from general and"
- [methods] install simpipe via devtools: "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [methods] simpipe2docker links R and Docker: "use [**simpipe2docker**](https://github.com/duohongrui/simpipe2docker) package for linking R environmrnt and the Docker container"
- [abstract] top methods under corresponding scenarios: "Lun, SPARSim, and scDesign3-tree outperform other methods under corresponding simulation scenarios"
- [discussion] data distortion caveat: "Some simulation algorithms may cause data distortion and over-simulation when generating datasets with ground truth."
