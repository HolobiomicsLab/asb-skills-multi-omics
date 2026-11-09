---
name: clustering-quality-metric-computation
description: Use when scoring simulated scRNA-seq datasets that carry ground-truth
  cell-group labels against clustering-quality metrics (ASW, Dunn, Connectivity, DB,
  CDI) to rank simulation methods.
when_to_use_negative:
- Simulated data carry no ground-truth cluster labels - batch-effect or trajectory
  scenarios require their own metric sets (CMS/LISI/MM/entropy/kBET/ASW-batch/PCR
  or HIM/F1branches/F1milestones/Cordist) instead of clustering-quality metrics
- The goal is accuracy assessment (distributional fidelity of simulated vs. real data,
  where SRTsim, scDesign3, ZINB-WaVE, and scDesign2 excel) rather than functionality
  scoring with ground truth
- The task is scalability benchmarking (where Phenopath, Lun, Simple, and MFA score
  high but cannot generate realistic data) rather than quantifying clustering fidelity
  of the simulated ground truth
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: simpipe
  role: Executes the estimation-simulation-evaluation pipeline; computes the functional
    metrics including the clustering-quality metrics (ASW, Dunn, Connectivity, DB,
    CDI) on simulated ground-truth data and aggregates per-scenario functionality
    scores
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Provides the 49 collected simulation methods that generate the simulated datasets
    with ground-truth cell-group annotations on which the clustering-quality metrics
    are computed
  repo: https://github.com/duohongrui/simmethods
- name: tislingshot
  role: Prerequisite install for simpipe when simulating datasets with trajectory
    information; enables the Slingshot-based trajectory branch (HIM, F1branches, F1milestones,
    Cordist) of the evaluation
  repo: https://github.com/dynverse/ti_slingshot
- name: simpipe2docker
  role: Links the R environment to the simpipe Docker container so the estimation-simulation-evaluation
    pipeline and metric computation run reproducibly
  repo: https://github.com/duohongrui/simpipe2docker
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:cb1b898f94974554d769c799f59b84ce9af3bb43f1cef30ca12a7963a10b1add
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# clustering-quality-metric-computation

## Summary

Computes clustering-quality metrics (ASW, Dunn, Connectivity, DB, CDI) on simulated scRNA-seq datasets carrying ground-truth cell-group annotations to score simulation-method functionality, as implemented in the simpipe pipeline of Duo et al. 2024 (Genome Biology).

## When to use

Use when scoring simulated scRNA-seq datasets that carry ground-truth cell-group labels against clustering-quality metrics (ASW, Dunn, Connectivity, DB, CDI) to rank simulation methods.

## When NOT to use

- Simulated data carry no ground-truth cluster labels - batch-effect or trajectory scenarios require their own metric sets (CMS/LISI/MM/entropy/kBET/ASW-batch/PCR or HIM/F1branches/F1milestones/Cordist) instead of clustering-quality metrics
- The goal is accuracy assessment (distributional fidelity of simulated vs. real data, where SRTsim, scDesign3, ZINB-WaVE, and scDesign2 excel) rather than functionality scoring with ground truth
- The task is scalability benchmarking (where Phenopath, Lun, Simple, and MFA score high but cannot generate realistic data) rather than quantifying clustering fidelity of the simulated ground truth

## Inputs

- Real scRNA-seq/SRT reference datasets (the study used 152 reference datasets from 24 platforms) from which simpipe estimates parameters
- Simulated single-cell datasets with ground-truth cell-group/DEG annotations, generated per method by simmethods via simpipe
- Scenario specification (cell-groups, DEGs, cell-batches, trajectory) determining which metric set is computed

## Outputs

- Per-dataset clustering-quality metric values (ASW, Dunn, Connectivity, DB, CDI, ROUGE for the cell-groups/DEG scenarios)
- Aggregated per-method functionality score for each simulation scenario
- functionality_scores.csv containing all computed scores

## How to apply

Install the evaluation stack via devtools (duohongrui/simmethods and duohongrui/simpipe; add dynverse/ti_slingshot only if trajectory data will be simulated) and generate simulated datasets with ground-truth annotations for each evaluated method (e.g., Lun, SPARSim, scMultiSim, Lun2) through the simpipe estimation-simulation-evaluation pipeline. For the cell-groups and DEG functionality scenarios, compute the clustering-quality metrics ASW, Dunn, Connectivity, DB, and CDI (plus ROUGE) on each simulated dataset; batch scenarios instead require CMS, LISI, MM, entropy, kBET, ASW-batch, and PCR, and trajectory scenarios require Slingshot-based HIM, F1branches, F1milestones, and Cordist. Aggregate each method's metric results into a per-scenario functionality score and write all scores to functionality_scores.csv. Judge correctness by reproducing the reported values: Lun 0.71 (cell groups) and 0.90 (DEGs), SPARSim 0.77 (cell batches), scMultiSim 0.56, Lun2 0.53, and by confirming that Class1 methods perform poorly on F1milestones and Cordist. Optionally run the stack through the simpipe Docker image via simpipe2docker and consult the Simsite documentation; note that a high functionality score does not guarantee realistic data, since Lun ignores sparsity and biological variability.

## Related tools

- **simpipe** (Executes the estimation-simulation-evaluation pipeline; computes the functional metrics including the clustering-quality metrics (ASW, Dunn, Connectivity, DB, CDI) on simulated ground-truth data and aggregates per-scenario functionality scores) — https://github.com/duohongrui/simpipe
- **simmethods** (Provides the 49 collected simulation methods that generate the simulated datasets with ground-truth cell-group annotations on which the clustering-quality metrics are computed) — https://github.com/duohongrui/simmethods
- **tislingshot** (Prerequisite install for simpipe when simulating datasets with trajectory information; enables the Slingshot-based trajectory branch (HIM, F1branches, F1milestones, Cordist) of the evaluation) — https://github.com/dynverse/ti_slingshot
- **simpipe2docker** (Links the R environment to the simpipe Docker container so the estimation-simulation-evaluation pipeline and metric computation run reproducibly) — https://github.com/duohongrui/simpipe2docker

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- Reproduced functionality scores match reported values: Lun 0.71 (cell groups) and 0.90 (DEGs); SPARSim 0.77 (cell batches); scMultiSim 0.56; Lun2 0.53
- Class1 simulation methods show poor performance on F1milestones and Cordist in the trajectory scenario
- The metric set matches the scenario: ASW/Dunn/Connectivity/DB/CDI/ROUGE for cell-groups and DEGs, not the batch (CMS/LISI/MM/entropy/kBET/ASW-batch/PCR) or trajectory (HIM/F1branches/F1milestones/Cordist) metrics
- All aggregated per-scenario scores are written to functionality_scores.csv, one score per evaluated method per scenario
- No missing or infinite values appear in metric calculations (execution errors in the study were mainly caused by failed parameter estimations and missing/infinite values)

## Limitations

- Methods relying on customized parameters (Splat, SCRIP, Lun, scMultiSim) may cause data distortion and over-simulation when generating ground-truth datasets, weakening the reliability of the computed functionality scores
- Lun, the top functionality performer, does not account for the sparsity and biological variability of scRNA-seq data, so a high clustering-quality score does not guarantee realistic data
- Execution errors are mainly caused by failed parameter estimations and the appearance of missing or infinite values in calculations

## Evidence

- [abstract] Lun, SPARSim, and scDesign3-tree outperform other methods under corresponding simulation scenarios: "Lun, SPARSim, and scDesign3-tree outperform other methods under corresponding simulation scenarios"
- [abstract] We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability: "We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability"
- [methods] Install simmethods and simpipe from GitHub using devtools: [evidence span withheld — anchor status: not_found]
- [readme] estimating parameters from real datasets, simulating single-cell RNA-seq datasets and evaluating the simulated data from general and functional perspectives: "estimating parameters from real datasets, simulating single-cell RNA-seq datasets and evaluating the simulated data from general and functional perspectives"
- [discussion] Some simulation algorithms may cause data distortion and over-simulation when generating datasets with ground truth.: "Some simulation algorithms may cause data distortion and over-simulation when generating datasets with ground truth."
- [discussion] However, it does not account for the sparsity and biological variability of scRNA-seq data.: "However, it does not account for the sparsity and biological variability of scRNA-seq data."
