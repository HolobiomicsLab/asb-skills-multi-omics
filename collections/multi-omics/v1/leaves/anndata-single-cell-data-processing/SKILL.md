---
name: anndata-single-cell-data-processing
description: Use when an annotated scRNA-seq AnnData (.h5ad) with cell-type labels
  needs preprocessing and exclusion of sparse label classes (e.g. 88/11,990 PBMC megakaryocytes)
  before annotation-trainability analysis.
when_to_use_negative:
- Input is convolved or aggregated data, such as that obtained from certain spatial
  transcriptomics technologies — Annotatability does not directly handle such data
- The input annotations are probabilistic or continuous rather than nominal categories,
  which Annotatability currently does not support
- The task is generic normalization or clustering of unannotated single-cell data
  with no label column to audit
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Annotatability
  role: Provides models.py and metrics.py that operate directly on the Scanpy-based
    AnnData to compute training dynamics and gene trainability scores
  repo: https://github.com/nitzanlab/Annotatability
- name: Scanpy
  role: AnnData container and single-cell preprocessing basis; Annotatability's code
    is built on the Scanpy package
  repo: https://scanpy.readthedocs.io/en/stable/
- name: PyTorch
  role: Backend for training the DNN (follow_training_dyn_neural_net) with device
    selection via torch.device('cuda:0' if torch.cuda.is_available() else 'cpu')
- name: numpy
  role: Converts detached confidence/variability tensors to numpy arrays stored in
    adata.obs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:4841828ef4441c8ff9a9f16ab311e6f65b1e0267d6797192c4aeeb359c849c1f
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
  - Python
  - Scanpy
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# anndata-single-cell-data-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Load and preprocess an annotated single-cell AnnData object (e.g. the PBMC scRNA-seq dataset with cell-type labels in adata.obs['label']) so that Annotatability can monitor DNN training dynamics and compute per-cell confidence/variability annotation-trainability scores.

## When to use

Use when an annotated scRNA-seq AnnData (.h5ad) with cell-type labels needs preprocessing and exclusion of sparse label classes (e.g. 88/11,990 PBMC megakaryocytes) before annotation-trainability analysis.

## When NOT to use

- Input is convolved or aggregated data, such as that obtained from certain spatial transcriptomics technologies — Annotatability does not directly handle such data
- The input annotations are probabilistic or continuous rather than nominal categories, which Annotatability currently does not support
- The task is generic normalization or clustering of unannotated single-cell data with no label column to audit

## Inputs

- Annotated AnnData (.h5ad) object with cell-type annotations in an observation column such as adata.obs['label'] (e.g. PBMC scRNA-seq, 11,990 cells)
- Filtering configuration (which low-count label classes to exclude, e.g. megakaryocytes)

## Outputs

- Preprocessed AnnData with excluded label classes and optionally L2-normalized (variance 1) gene expression
- AnnData with per-cell adata.obs['conf'] and adata.obs['var'] training-dynamics scores
- Ranked AnnData with adata_ranked.var['conf_score_high'] and adata_ranked.var['conf_score_low'] gene trainability scores
- Saved results file, e.g. pbmc_annotatability_results.h5ad

## How to apply

Load the annotated AnnData from the Annotatability_notebooks reproducibility repository, ensuring cell-type annotations are stored as an observation column (e.g. adata.obs['label']). Apply dataset-specific filtering steps documented in the paper — for the PBMC scRNA-seq dataset, exclude the 88 of 11,990 cells annotated as megakaryocytes due to their low number. As a preprocessing step for trainability association scoring, scale the expression level of each gene to variance 1 (L2 normalization), and retain only genes expressed in more than 3 cells and among highly variable genes (3,000 genes) where applicable. Keep the AnnData Scanpy-based so that Annotatability's models.py training functions (follow_training_dyn_neural_net) and metrics.py scoring functions (rank_genes_conf_min_counts, probability_list_to_confidence_and_var) can consume it directly, with results written back into adata.obs['conf'], adata.obs['var'], and adata_ranked.var['conf_score_high'/'conf_score_low'].

## Related tools

- **Annotatability** (Provides models.py and metrics.py that operate directly on the Scanpy-based AnnData to compute training dynamics and gene trainability scores) — https://github.com/nitzanlab/Annotatability
- **Scanpy** (AnnData container and single-cell preprocessing basis; Annotatability's code is built on the Scanpy package) — https://scanpy.readthedocs.io/en/stable/
- **PyTorch** (Backend for training the DNN (follow_training_dyn_neural_net) with device selection via torch.device('cuda:0' if torch.cuda.is_available() else 'cpu'))
- **numpy** (Converts detached confidence/variability tensors to numpy arrays stored in adata.obs)

## Examples

```
prob_list = models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=epoch_num, device=device); all_conf, all_var = models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=epoch_num)
```

## Evaluation signals

- adata.obs['conf'] equals the average probability assigned to the correct label across all epochs, and adata.obs['var'] the standard deviation of that probability
- Excluded label class matches the documented filter: 88 of 11,990 PBMC cells annotated as megakaryocytes removed
- Cells with high confidence/low variability (easy-to-learn) dominate the correctly annotated classes; hard-to-learn CD14+ monocytes underexpress CD14 while ~80% express IL7R and ~60% express NKG7
- adata_ranked.var contains both 'conf_score_high' and 'conf_score_low' columns and marker genes of CD14+ monocytes and B cells show negative trainability association
- Final AnnData saves without error via adata.write('pbmc_annotatability_results.h5ad')

## Limitations

- Thresholding on confidence and variability scores for categorizing introduces arbitrary cutoffs that could influence data interpretation
- Determination of hyperparameters such as training time needs standardization, as they may influence confidence and variability scores
- The percentile q for hard-to-learn thresholds is chosen dataset-dependently (25%–90%), a manual, non-standardized choice; no software version is reported for PyTorch and no changelog exists for the repository

## Evidence

- [discussion] Exclude PBMC cells annotated as megakaryocytes: "For the PBMC dataset, we excluded cells annotated as megakaryocytes owing to their low number (88/11,990)."
- [discussion] L2 normalization preprocessing: "As a preprocessing step, the expression level of each gene is scaled to have variance 1 (L2 normalization)."
- [methods] Training and scoring on AnnData via models.py: "prob_list = models.follow_training_dyn_neural_net(adata, label_key='label',iterNum=epoch_num, device=device)"
- [readme] Gene ranking output columns: "adata_ranked.var['conf_score_high'] %annotation-trainability positive association score"
- [methods] Scanpy-based code basis: "Our code is based on [Scanpy](https://scanpy.readthedocs.io/en/stable/) package"
