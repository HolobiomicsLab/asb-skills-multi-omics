---
name: scrna-seq-annotation-error-detection
description: Use when an annotated scRNA-seq AnnData (cell-type labels in adata.obs['label'])
  is suspected to contain mislabeled cells, and you want training-dynamics-based evidence
  plus marker-gene verification.
when_to_use_negative:
- Input is convolved or aggregated spatial transcriptomics data — the article states
  Annotatability 'does not directly handle convolved or aggregated data'
- Annotations are probabilistic or continuous — Annotatability 'currently treats input
  annotations as nominal categories only'
- A specific cell type has very few annotated cells (like the PBMC megakaryocytes,
  88/11,990), where training dynamics per class are unreliable — exclude such classes
  first
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Annotatability
  role: 'Core framework: trains the neural network (models.follow_training_dyn_neural_net),
    computes confidence/variability (models.probability_list_to_confidence_and_var),
    and ranks genes (metrics.rank_genes_conf_min_counts)'
  repo: https://github.com/nitzanlab/Annotatability
- name: Scanpy
  role: Base package for AnnData handling and visualization of the trainability-aware
    results (sc.tl.umap, sc.pl.umap)
  repo: https://scanpy.readthedocs.io/en/stable/
- name: PyTorch
  role: Backend used to perform the neural network training procedure and device selection
    (cuda:0 if available)
- name: Annotatability_notebooks
  role: Reproducibility repository providing the PBMC notebook, annotated data, and
    tutorials used to reproduce the Fig. 2 result
  repo: https://github.com/nitzanlab/Annotatability_notebooks
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:7a9905e3f42edd09dda1bdd1da27dcc4b8bcc35b0004e6e030c62ae54a02a1ff
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
  - Python
  - Scanpy
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# scrna-seq-annotation-error-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A skill for detecting erroneously annotated cells in annotated single-cell RNA-seq data (e.g., human PBMC) by monitoring the training dynamics of a multilayer perceptron with Annotatability, converting per-epoch prediction probabilities into confidence and variability scores, and verifying suspected mismatches against marker gene expression.

## When to use

Use when an annotated scRNA-seq AnnData (cell-type labels in adata.obs['label']) is suspected to contain mislabeled cells, and you want training-dynamics-based evidence plus marker-gene verification.

## When NOT to use

- Input is convolved or aggregated spatial transcriptomics data — the article states Annotatability 'does not directly handle convolved or aggregated data'
- Annotations are probabilistic or continuous — Annotatability 'currently treats input annotations as nominal categories only'
- A specific cell type has very few annotated cells (like the PBMC megakaryocytes, 88/11,990), where training dynamics per class are unreliable — exclude such classes first

## Inputs

- Annotated AnnData (.h5ad) scRNA-seq object with cell-type annotations stored in adata.obs['label'] (e.g., PBMC scRNA-seq, 11,990 cells)
- Marker gene lists for the annotated and suspected true cell types (e.g., CD14 for monocytes; IL7R, NKG7 for T/NK cells)

## Outputs

- adata.obs['conf']: per-cell average probability of the correct label across epochs
- adata.obs['var']: per-cell standard deviation of the correct-label probability across epochs
- Per-cell classification into easy-to-learn, hard-to-learn, or ambiguous categories
- adata_ranked.var['conf_score_high'] and adata_ranked.var['conf_score_low']: gene-level positive/negative annotation-trainability association scores
- pbmc_annotatability_results.h5ad: AnnData with confidence/variability scores, class labels, and gene trainability rankings

## How to apply

Load the annotated AnnData and exclude low-count cell-type classes (for PBMC, exclude cells annotated as megakaryocytes, 88/11,990 cells). Train the multilayer perceptron with models.follow_training_dyn_neural_net using label_key='label' and defaults (iterNum=100 epochs, lr=0.001, momentum=0.9, batch_size=256, weighted_sampler=True, num_layers=3) on cuda:0 if available, recording the predicted probability of the correct label after each epoch. Convert the probability list to per-cell scores with models.probability_list_to_confidence_and_var, storing adata.obs['conf'] (average probability of the correct label across epochs) and adata.obs['var'] (standard deviation across epochs). Classify cells into easy-to-learn (high confidence, low variability; likely correctly annotated), hard-to-learn (low confidence, low variability; likely erroneously annotated), or ambiguous (moderate confidence, high variability). Optionally rank genes by annotation-trainability association with metrics.rank_genes_conf_min_counts, yielding adata_ranked.var['conf_score_high'] and adata_ranked.var['conf_score_low'], and verify that markers of the purported cell type (e.g., CD14 for CD14+ monocytes) are negatively associated with trainability while markers of the true identity (e.g., IL7R, NKG7 for T/NK cells) are expressed. Save the annotated results as .h5ad.

## Related tools

- **Annotatability** (Core framework: trains the neural network (models.follow_training_dyn_neural_net), computes confidence/variability (models.probability_list_to_confidence_and_var), and ranks genes (metrics.rank_genes_conf_min_counts)) — https://github.com/nitzanlab/Annotatability
- **Scanpy** (Base package for AnnData handling and visualization of the trainability-aware results (sc.tl.umap, sc.pl.umap)) — https://scanpy.readthedocs.io/en/stable/
- **PyTorch** (Backend used to perform the neural network training procedure and device selection (cuda:0 if available))
- **Annotatability_notebooks** (Reproducibility repository providing the PBMC notebook, annotated data, and tutorials used to reproduce the Fig. 2 result) — https://github.com/nitzanlab/Annotatability_notebooks

## Examples

```
prob_list = models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=100, device=device); all_conf, all_var = models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=100)
```

## Evaluation signals

- Hard-to-learn CD14+ monocytes underexpress CD14, while ~80% express the T cell marker IL7R and ~60% express the NK/T cell marker NKG7, confirming annotation mismatch
- Marker genes of the annotated cell types (CD14+ monocytes, B cells) are negatively associated with annotation trainability (conf_score_low), reproducing the Fig. 2 result
- Correct-label probability is recorded for every epoch for every cell, and conf/var arrays have length adata.n_obs
- Cells with high confidence and low variability dominate the easy-to-learn class for datasets with a relatively low fraction of erroneously annotated cells (like PBMC)
- Final AnnData contains obs['conf'], obs['var'], class labels, and var['conf_score_high']/'conf_score_low' and saves without errors to .h5ad

## Limitations

- Thresholding on confidence and variability scores introduces arbitrary cutoffs that could influence data interpretation
- Determination of hyperparameters such as training time (iterNum) needs standardization, as they may influence the confidence and variability scores
- The percentile q for hard-to-learn thresholds is chosen dataset-dependently (25%–90%), a manual, non-standardized choice
- No software version is reported for PyTorch, limiting reproducibility of the training procedure

## Evidence

- [discussion] Confidence is defined as the average probability assigned by the model to the correct label across all epochs; variability is the standard deviation of that probability across epochs.: [evidence span withheld — anchor status: not_found]
