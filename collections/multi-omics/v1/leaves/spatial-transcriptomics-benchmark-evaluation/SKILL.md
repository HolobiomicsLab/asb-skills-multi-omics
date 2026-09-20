---
name: spatial-transcriptomics-benchmark-evaluation
description: Use when auditing cell-type annotations in spatial (e.g. MERFISH) single-cell
  data and needing a quantitative AUROC comparison of mislabel-detection methods over
  controlled synthetic mislabeling fractions.
when_to_use_negative:
- Annotatability does not directly handle convolved or aggregated spatial transcriptomics
  data, so aggregated spatial measurements are a poor fit without adaptation
- The input annotations are probabilistic or continuous rather than nominal categories,
  since Annotatability currently treats input annotations as nominal categories only
- 'You need a fully standardized hyperparameter-free benchmark: hyperparameters such
  as training time influence the confidence and variability scores and are not yet
  standardized'
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Annotatability
  role: Provides follow_training_dyn_neural_net and probability_list_to_confidence_and_var
    to produce per-cell confidence/variability scores used for mislabel detection
  repo: https://github.com/nitzanlab/Annotatability
- name: PyTorch
  role: Backend for DNN training of the neural network used in follow_training_dyn_neural_net,
    on cuda device if available
- name: Scanpy
  role: AnnData handling and visualization (sc.tl.umap, sc.pl.umap) of the annotated
    spatial data
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scReClassify
  role: Baseline mislabel-detection method scored with SVM, KNN, and RF base classifiers
    for AUROC comparison
- name: numpy
  role: Numerical computation for perturbing labels and computing AUROC metrics
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:1a58eaad7b335d288f1229f6a13a62905b08f8fa51e5273b2b34186ddf56c321
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
  - PyTorch
  - Scanpy
  - numpy
  - pandas
  - scReClassify
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# spatial-transcriptomics-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark Annotatability's training-dynamics mislabel detection against scReClassify-based baselines (SVM, KNN, RF) on spatial transcriptomics data by building semi-synthetically perturbed annotations and comparing AUROC of mislabel detection across mislabeling fractions.

## When to use

Use when auditing cell-type annotations in spatial (e.g. MERFISH) single-cell data and needing a quantitative AUROC comparison of mislabel-detection methods over controlled synthetic mislabeling fractions.

## When NOT to use

- Annotatability does not directly handle convolved or aggregated spatial transcriptomics data, so aggregated spatial measurements are a poor fit without adaptation
- The input annotations are probabilistic or continuous rather than nominal categories, since Annotatability currently treats input annotations as nominal categories only
- You need a fully standardized hyperparameter-free benchmark: hyperparameters such as training time influence the confidence and variability scores and are not yet standardized

## Inputs

- Annotated MERFISH mouse hypothalamic preoptic region AnnData object (.h5ad) with cell-type labels in an observation column
- Ground-truth correct/incorrect flag per cell (for AUROC computation)
- List of synthetic mislabeling fractions (e.g. 5%-50%)

## Outputs

- Per-method AUROC values for mislabel detection at each mislabeling percentage
- adata.obs['conf'] and adata.obs['var'] per-cell confidence and variability scores
- Figure auroc_vs_mislabeled_fraction.png with one AUROC curve per method across all perturbation levels

## How to apply

Load the annotated MERFISH mouse hypothalamic preoptic region AnnData object from the Annotatability_notebooks reproducibility repository with the cell-type label stored as an observation column. Create semi-synthetic perturbed copies by randomly reassigning the 'label' observation of controlled fractions of cells (5%-50% mislabeled) while keeping a per-cell ground-truth correct/incorrect flag. For each perturbation level, run models.follow_training_dyn_neural_net with iterNum=50 epochs, lr=0.001, momentum=0.9, batch_size=256, weighted_sampler=True, num_layers=3 on GPU if available, then compute per-cell confidence and variability with models.probability_list_to_confidence_and_var. Score the same perturbed datasets with scReClassify using SVM, KNN, and RF base classifiers to obtain per-cell mislabel predictions, then compute the AUROC of mislabel detection (using the correct/incorrect flag as ground truth) for every method at every mislabeling percentage and plot AUROC versus percentage of mislabeled cells.

## Related tools

- **Annotatability** (Provides follow_training_dyn_neural_net and probability_list_to_confidence_and_var to produce per-cell confidence/variability scores used for mislabel detection) — https://github.com/nitzanlab/Annotatability
- **PyTorch** (Backend for DNN training of the neural network used in follow_training_dyn_neural_net, on cuda device if available)
- **Scanpy** (AnnData handling and visualization (sc.tl.umap, sc.pl.umap) of the annotated spatial data) — https://scanpy.readthedocs.io/en/stable/
- **scReClassify** (Baseline mislabel-detection method scored with SVM, KNN, and RF base classifiers for AUROC comparison)
- **numpy** (Numerical computation for perturbing labels and computing AUROC metrics)

## Examples

```
from Annotatability import models; prob_list = models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=50, lr=0.001, momentum=0.9, batch_size=256, weighted_sampler=True, num_layers=3, device=torch.device('cuda:0' if torch.cuda.is_available() else 'cpu'))
```

## Evaluation signals

- The figure auroc_vs_mislabeled_fraction.png exists and contains one AUROC curve per method (Annotatability plus SVM, KNN, RF baselines) across all perturbation levels
- Annotatability's AUROC values are computed from confidence/variability scores stored in adata.obs['conf'] and adata.obs['var']
- Ground truth for AUROC is the per-cell correct/incorrect flag retained from the semi-synthetic perturbation
- The expected result is that Annotatability consistently outperforms the scReClassify-based baselines, with the advantage growing as more cells are synthetically misclassified (5%-50%)
- Reproduction uses the documented hyperparameters: iterNum=50, lr=0.001, momentum=0.9, batch_size=256, weighted_sampler=True, num_layers=3

## Limitations

- Hyperparameters such as training time influence the confidence and variability scores and need standardization, which may affect benchmark comparability
- Thresholding on confidence and variability scores introduces arbitrary cutoffs that could influence interpretation
- Annotatability treats input annotations as nominal categories only and does not directly handle convolved or aggregated spatial transcriptomics data
- Percentile q for hard-to-learn thresholds is chosen dataset-dependently (25%-90%), a manual, non-standardized choice

## Evidence

- [discussion] we define confidence, $$\widehat{{u}_{i}}$$, as the average probability assigned by the model to the correct label: [evidence span withheld — anchor status: not_found]
