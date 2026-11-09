---
name: semi-synthetic-label-perturbation
description: 'Use when auditing a cell-annotation error detector: you need a ground-truth
  correct/incorrect flag per cell to compute AUROC, so labels must be synthetically
  misclassified at controlled fractions.'
when_to_use_negative:
- Annotatability does not directly handle convolved or aggregated data, such as that
  obtained from certain spatial transcriptomics technologies — verify the data are
  per-cell, not aggregated
- You lack externally assigned annotations to perturb; semi-synthetic error injection
  requires a nominal 'label' observation column to reassign
- The goal is reannotation of real suspected errors rather than controlled benchmarking;
  use the reannotation module instead of perturbation-based AUROC evaluation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: Annotatability
  role: Trains a DNN on perturbed data via models.follow_training_dyn_neural_net and
    derives per-cell confidence/variability scores for mislabel AUROC
  repo: https://github.com/nitzanlab/Annotatability
- name: PyTorch
  role: Backend for the DNN training procedure and device selection (cuda:0 if available)
- name: Scanpy
  role: AnnData handling and codebase foundation for loading and manipulating the
    annotated single-cell/spatial data
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scReClassify
  role: Provides baseline mislabel-detection classifiers (SVM, KNN, RF) scored on
    the same perturbed datasets
- name: Annotatability_notebooks
  role: Reproducibility repository supplying the annotated MERFISH dataset and benchmark
    workflows
  repo: https://github.com/nitzanlab/Annotatability_notebooks
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:376582a08713a336df682b5e1085acadc531fb7deaa45cfc7e9b914f2522c5cc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/nitzanlab/Annotatability
  repo_url: https://github.com/nitzanlab/Annotatability
  tools:
  - Annotatability
  - Annotatability_notebooks
  - PyTorch
  - Scanpy
  - numpy
  - pandas
  - scReClassify
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# semi-synthetic-label-perturbation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a mislabel-detection method by creating semi-synthetic perturbed copies of an annotated dataset (e.g. MERFISH mouse hypothalamic preoptic region AnnData) in which controlled fractions (5%–50%) of cell-type labels are randomly reassigned, then scoring AUROC of correct-vs-erroneous detection against baselines.

## When to use

Use when auditing a cell-annotation error detector: you need a ground-truth correct/incorrect flag per cell to compute AUROC, so labels must be synthetically misclassified at controlled fractions.

## When NOT to use

- Annotatability does not directly handle convolved or aggregated data, such as that obtained from certain spatial transcriptomics technologies — verify the data are per-cell, not aggregated
- You lack externally assigned annotations to perturb; semi-synthetic error injection requires a nominal 'label' observation column to reassign
- The goal is reannotation of real suspected errors rather than controlled benchmarking; use the reannotation module instead of perturbation-based AUROC evaluation

## Inputs

- Annotated AnnData object (e.g. MERFISH mouse hypothalamic preoptic region) with cell-type labels in an adata.obs column (label_key='label')
- List of mislabeling fractions to test (e.g. 5%–50% of cells)
- Baseline classifiers for comparison (scReClassify with SVM, KNN, RF)

## Outputs

- Semi-synthetic perturbed AnnData copies with a per-cell correct/incorrect ground-truth flag
- Per-cell confidence (adata.obs['conf']) and variability (adata.obs['var']) scores from training dynamics
- AUROC values per method at each perturbation level
- Figure plotting AUROC versus percentage of mislabeled cells (e.g. auroc_vs_mislabeled_fraction.png)

## How to apply

Load an annotated AnnData object (e.g. MERFISH mouse hypothalamic preoptic region data with cell-type labels in an observation column) and create perturbed copies by randomly reassigning the 'label' observation of controlled fractions of cells (5%–50%) while keeping a per-cell correct/incorrect ground-truth flag. For each perturbation level, run the method under test — here Annotatability's models.follow_training_dyn_neural_net (iterNum=50 epochs, lr=0.001, momentum=0.9, batch_size=256, weighted_sampler=True, num_layers=3, GPU if available) — then compute per-cell confidence and variability via models.probability_list_to_confidence_and_var and store them in adata.obs['conf'] and adata.obs['var']. Score the same perturbed datasets with scReClassify baselines (SVM, KNN, RF) to obtain per-cell mislabel predictions. Compute AUROC of mislabel detection using the correct/incorrect flag as ground truth for every method at every mislabeling percentage, and plot AUROC versus the percentage of mislabeled cells. The rationale is that semi-synthetic perturbation provides known error locations, which real annotations lack, enabling quantitative AUROC comparison as mislabeling burden grows.

## Related tools

- **Annotatability** (Trains a DNN on perturbed data via models.follow_training_dyn_neural_net and derives per-cell confidence/variability scores for mislabel AUROC) — https://github.com/nitzanlab/Annotatability
- **PyTorch** (Backend for the DNN training procedure and device selection (cuda:0 if available))
- **Scanpy** (AnnData handling and codebase foundation for loading and manipulating the annotated single-cell/spatial data) — https://scanpy.readthedocs.io/en/stable/
- **scReClassify** (Provides baseline mislabel-detection classifiers (SVM, KNN, RF) scored on the same perturbed datasets)
- **Annotatability_notebooks** (Reproducibility repository supplying the annotated MERFISH dataset and benchmark workflows) — https://github.com/nitzanlab/Annotatability_notebooks

## Examples

```
from Annotatability import metrics, models; prob_list = models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=50, lr=0.001, momentum=0.9, batch_size=256, weighted_sampler=True, num_layers=3, device=torch.device('cuda:0' if torch.cuda.is_available() else 'cpu')); all_conf, all_var = models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=50)
```

## Evaluation signals

- Figure auroc_vs_mislabeled_fraction.png exists and contains one AUROC curve per method (Annotatability, SVM, KNN, RF) across all perturbation levels (5%–50%)
- Annotatability's AUROC is computed from scores stored in adata.obs['conf'] and adata.obs['var'] produced by probability_list_to_confidence_and_var with epoch_num=50
- Ground-truth labels for AUROC come from the per-cell correct/incorrect flag preserved during perturbation, not the reassigned 'label' column
- Annotatability's AUROC consistently exceeds the scReClassify baselines, with the advantage growing as more cells are synthetically misclassified
- Training completed on the intended device (cuda:0 if GPU available) with batch_size=256 and weighted_sampler=True

## Limitations

- Determination of hyperparameters such as training time (iterNum) needs standardization, as they may influence the confidence and variability scores and hence AUROC
- Thresholding on confidence and variability scores introduces arbitrary cutoffs that could influence interpretation of detection results
- Percentile q for hard-to-learn thresholds is chosen dataset-dependently (25%–90%), a manual, non-standardized choice
- Semi-synthetic errors are random reassignments and may not reflect the structure of real annotation mistakes

## Evidence
