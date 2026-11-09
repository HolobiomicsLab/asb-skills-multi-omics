---
name: marker-gene-trainability-ranking
description: Use when an AnnData object already carries per-cell confidence ('conf')
  and variability ('var') scores from DNN training dynamics and you need to rank genes
  by positive or negative association with annotation trainability, e.g. to verify
  erroneous annotations via marker gene expression.
when_to_use_negative:
- Per-cell confidence and variability scores have not yet been computed — run the
  DNN training-dynamics step (follow_training_dyn_neural_net plus probability_list_to_confidence_and_var)
  first
- Data are convolved or aggregated, such as that obtained from certain spatial transcriptomics
  technologies, which Annotatability does not directly handle
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Annotatability
  role: Provides metrics.rank_genes_conf_min_counts which scores and ranks genes by
    annotation-trainability association
  repo: https://github.com/nitzanlab/Annotatability
- name: Scanpy
  role: Underlying AnnData framework holding expression matrix and obs/var slots where
    scores are stored
  repo: https://scanpy.readthedocs.io/en/stable/
- name: PyTorch
  role: Performs the DNN training whose dynamics produce the confidence scores the
    gene ranking is computed against
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:ed8e478a8ec37aca8f731f3a330c21108a6e69b80bea41776bd8d412c1a692dc
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# Marker-gene trainability ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores and ranks genes by their association with annotation trainability (confidence of DNN training dynamics) in single-cell data, identifying markers positively or negatively associated with correctly annotated cells using Annotatability's rank_genes_conf_min_counts.

## When to use

Use when an AnnData object already carries per-cell confidence ('conf') and variability ('var') scores from DNN training dynamics and you need to rank genes by positive or negative association with annotation trainability, e.g. to verify erroneous annotations via marker gene expression.

## When NOT to use

- Per-cell confidence and variability scores have not yet been computed — run the DNN training-dynamics step (follow_training_dyn_neural_net plus probability_list_to_confidence_and_var) first
- Data are convolved or aggregated, such as that obtained from certain spatial transcriptomics technologies, which Annotatability does not directly handle

## Inputs

- Scanpy AnnData object with scRNA-seq expression matrix
- adata.obs['conf'] — per-cell average probability of the correct label across training epochs
- adata.obs['var'] — per-cell standard deviation of that probability across epochs
- Optionally a cell label observation (e.g. adata.obs['label']) used upstream for training

## Outputs

- adata_ranked.var['conf_score_high'] — annotation-trainability positive association score per gene
- adata_ranked.var['conf_score_low'] — annotation-trainability negative association score per gene
- Gene ranking distinguishing markers of correctly annotated vs erroneously annotated cell populations

## How to apply

First ensure adata.obs['conf'] (average probability of the correct label across epochs) and adata.obs['var'] (standard deviation across epochs) are populated by running models.follow_training_dyn_neural_net followed by models.probability_list_to_confidence_and_var. As a preprocessing step, scale the expression level of each gene to variance 1 (L2 normalization) before computing association scores. Then call metrics.rank_genes_conf_min_counts(adata), which returns adata_ranked.var['conf_score_high'] for genes positively associated with annotation trainability and adata_ranked.var['conf_score_low'] for negative association. Rationale: genes whose expression tracks the confidence signal (e.g. underexpressed CD14 in erroneously annotated CD14+ monocytes that instead express IL7R and NKG7) reveal annotation mismatches or biologically meaningful states like healthy-like β cells in mSTZ data.

## Related tools

- **Annotatability** (Provides metrics.rank_genes_conf_min_counts which scores and ranks genes by annotation-trainability association) — https://github.com/nitzanlab/Annotatability
- **Scanpy** (Underlying AnnData framework holding expression matrix and obs/var slots where scores are stored) — https://scanpy.readthedocs.io/en/stable/
- **PyTorch** (Performs the DNN training whose dynamics produce the confidence scores the gene ranking is computed against)

## Examples

```
from Annotatability import metrics; adata_ranked = metrics.rank_genes_conf_min_counts(adata); scores_high = adata_ranked.var['conf_score_high']; scores_low = adata_ranked.var['conf_score_low']
```

## Evaluation signals

- adata_ranked.var contains both 'conf_score_high' and 'conf_score_low' columns with one value per gene
- For the PBMC case, CD14 (CD14+ monocyte marker) and B-cell markers should be negatively associated with trainability (underexpressed in erroneously annotated cells), while IL7R (~80% of erroneous CD14+ cells) and NKG7 (~60%) should be positively associated
- For mSTZ β cells, conf_score_high/conf_score_low should separate marker-expression patterns distinguishing disease, intermediate and healthy-like Louvain communities
- Ranking should be reproducible after L2 normalization (each gene scaled to variance 1) as prescribed preprocessing

## Limitations

- Thresholding on confidence and variability scores for categorizing cells introduces arbitrary cutoffs that could influence the interpretation of the data
- The percentile q for hard-to-learn thresholds is chosen dataset-dependently (25%–90%), a manual, non-standardized choice that can affect downstream gene rankings
- Hyperparameters such as training time (iterNum, default 100 epochs) are not standardized and may influence the confidence and variability scores feeding the ranking

## Evidence

- [discussion] Confidence and variability definitions: [evidence span withheld — anchor status: not_found]
