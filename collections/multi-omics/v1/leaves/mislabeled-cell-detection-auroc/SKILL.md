---
name: mislabeled-cell-detection-auroc
description: Use when auditing cell-type annotations of single-cell or spatial transcriptomics
  data (e.g. MERFISH .h5ad with a 'label' obs column) and you need to quantify, via
  AUROC, how well confidence/variability training-dynamics scores detect synthetically
  mislabeled cells versus classifier-based baselines.
when_to_use_negative:
- Annotatability does not directly handle convolved or aggregated data, such as that
  obtained from certain spatial transcriptomics technologies — deconvolve or use cell-level
  data first
- The input annotations are probabilistic or continuous rather than nominal categories,
  since Annotatability currently treats input annotations as nominal categories only
- 'You need a fully standardized, parameter-free audit: hyperparameters such as training
  time (iterNum) influence the confidence and variability scores and are not yet standardized'
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Annotatability
  role: Trains a DNN on annotated single-cell data and derives per-cell confidence
    and variability from training dynamics for mislabel detection
  repo: https://github.com/nitzanlab/Annotatability
- name: PyTorch
  role: Backend used to perform the DNN training procedure (device selection, weighted
    sampler, DataLoader)
- name: Scanpy
  role: Base single-cell framework for handling the AnnData object and visualization
    (e.g. sc.tl.umap, sc.pl.umap)
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scReClassify
  role: Baseline framework scored with SVM, KNN and RF base classifiers to produce
    per-cell mislabel predictions for AUROC comparison
- name: Annotatability_notebooks
  role: Reproducibility repository providing the annotated MERFISH benchmark data
    and benchmarking notebooks
  repo: https://github.com/nitzanlab/Annotatability_notebooks
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:478cc3284978aba2601a1997043a993b0229e33d883263081fafc2d6c1777114
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# mislabeled-cell-detection-auroc

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks Annotatability's DNN training-dynamics confidence/variability scores against scReClassify-based baselines (SVM, KNN, RF) for detecting erroneously annotated cells, using AUROC on semi-synthetically perturbed annotations (e.g. MERFISH mouse hypothalamic preoptic region data) across mislabeling fractions.

## When to use

Use when auditing cell-type annotations of single-cell or spatial transcriptomics data (e.g. MERFISH .h5ad with a 'label' obs column) and you need to quantify, via AUROC, how well confidence/variability training-dynamics scores detect synthetically mislabeled cells versus classifier-based baselines.

## When NOT to use

- Annotatability does not directly handle convolved or aggregated data, such as that obtained from certain spatial transcriptomics technologies — deconvolve or use cell-level data first
- The input annotations are probabilistic or continuous rather than nominal categories, since Annotatability currently treats input annotations as nominal categories only
- You need a fully standardized, parameter-free audit: hyperparameters such as training time (iterNum) influence the confidence and variability scores and are not yet standardized

## Inputs

- Annotated AnnData (.h5ad) object, e.g. MERFISH mouse hypothalamic preoptic region data with cell-type labels in an adata.obs column
- Semi-synthetic perturbed copies with controlled mislabeling fractions (5%–50%) and a per-cell correct/incorrect ground-truth flag
- scReClassify with SVM, KNN and RF base classifiers

## Outputs

- Per-cell confidence and variability scores stored in adata.obs['conf'] and adata.obs['var']
- Per-cell mislabel predictions from scReClassify (SVM, KNN, RF)
- AUROC of mislabel detection per method at each mislabeling percentage
- Figure auroc_vs_mislabeled_fraction.png with one AUROC curve per method across all perturbation levels

## How to apply

Load the annotated AnnData object (cell-type label stored as an observation column) from the Annotatability_notebooks reproducibility repository, then create semi-synthetic perturbed copies by randomly reassigning the 'label' of controlled fractions of cells (e.g. 5%–50% mislabeled) while keeping a per-cell correct/incorrect ground-truth flag. For each perturbation level, run models.follow_training_dyn_neural_net with iterNum=50 epochs, lr=0.001, momentum=0.9, batch_size=256, weighted_sampler=True, num_layers=3 on GPU if available, then compute per-cell confidence and variability via models.probability_list_to_confidence_and_var and store them in adata.obs['conf'] and adata.obs['var']. Score the same perturbed datasets with scReClassify using SVM, KNN and RF base classifiers to obtain per-cell mislabel predictions. Compute AUROC of mislabel detection (correct/incorrect flag as ground truth) for Annotatability and each scReClassify classifier at every mislabeling percentage, then plot AUROC versus percentage of mislabeled cells. The rationale is that confidence (average probability assigned to the correct label across epochs) and variability (its standard deviation) encode a 'goodness-of-match' signal that is oblivious to how annotations were assigned externally, enabling fair comparison with classifier-based mislabel detectors.

## Related tools

- **Annotatability** (Trains a DNN on annotated single-cell data and derives per-cell confidence and variability from training dynamics for mislabel detection) — https://github.com/nitzanlab/Annotatability
- **PyTorch** (Backend used to perform the DNN training procedure (device selection, weighted sampler, DataLoader))
- **Scanpy** (Base single-cell framework for handling the AnnData object and visualization (e.g. sc.tl.umap, sc.pl.umap)) — https://scanpy.readthedocs.io/en/stable/
- **scReClassify** (Baseline framework scored with SVM, KNN and RF base classifiers to produce per-cell mislabel predictions for AUROC comparison)
- **Annotatability_notebooks** (Reproducibility repository providing the annotated MERFISH benchmark data and benchmarking notebooks) — https://github.com/nitzanlab/Annotatability_notebooks

## Examples

```
prob_list = models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=50, device=device); all_conf, all_var = models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=50); adata.obs['conf'] = all_conf.detach().numpy(); adata.obs['var'] = all_var.detach().numpy()
```

## Evaluation signals

- The figure auroc_vs_mislabeled_fraction.png exists and contains one AUROC curve per method (Annotatability, SVM, KNN, RF) across all perturbation levels
- Annotatability's AUROC values are computed from confidence/variability scores stored in adata.obs['conf'] and adata.obs['var']
- Annotatability consistently outperforms the scReClassify-based baselines (SVM, KNN, RF), with the advantage growing as more cells are synthetically misclassified
- Each perturbation level uses a ground-truth correct/incorrect flag per cell as AUROC labels, covering the 5%–50% mislabeling range
- Training used the specified hyperparameters: iterNum=50, lr=0.001, momentum=0.9, batch_size=256, weighted_sampler=True, num_layers=3

## Limitations

- Annotatability does not directly handle convolved or aggregated data, such as that obtained from certain spatial transcriptomics technologies
- Determination of hyperparameters such as training time needs standardization, as they may influence the confidence and variability scores
- The semi-synthetic setting relies on arbitrary thresholding/percentile cutoffs (q chosen dataset-dependently between 25% and 90%) that could influence interpretation

## Evidence

- [discussion] we define confidence, $$\widehat{{u}_{i}}$$, as the average probability assigned by the model to the correct label: [evidence span withheld — anchor status: not_found]
