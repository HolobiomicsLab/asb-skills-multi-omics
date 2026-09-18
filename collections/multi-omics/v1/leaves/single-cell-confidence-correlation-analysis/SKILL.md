---
name: single-cell-confidence-correlation-analysis
description: Use when you have an AnnData object with per-cell confidence scores from
  DNN training dynamics (e.g., mSTZ β cells annotated by condition) and want to rank
  genes by annotation-trainability association.
when_to_use_negative:
- The AnnData lacks per-cell confidence scores — run follow_training_dyn_neural_net
  and probability_list_to_confidence_and_var first, since ranking correlates expression
  with the confidence vector
- Input is aggregated/convolved spatial transcriptomics data — Annotatability does
  not directly handle convolved or aggregated data
- The goal is differential expression between arbitrary external labels rather than
  association with annotation trainability (per-cell confidence from DNN training
  dynamics)
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Annotatability
  role: Provides metrics.rank_genes_conf_min_counts, the gene-ranking function that
    computes conf_score_high/conf_score_low by correlating scaled gene expression
    with the per-cell confidence vector
  repo: https://github.com/nitzanlab/Annotatability
- name: Scanpy
  role: Base framework for handling the AnnData object and single-cell preprocessing/scaling
  repo: https://scanpy.readthedocs.io/en/stable/
- name: pandas
  role: Dependency used to store/export the ranked gene table (e.g., mstz_gene_trainability_scores.csv)
- name: numpy
  role: Numerical dependency of Annotatability used in the correlation computations
- name: scipy
  role: Dependency of Annotatability supporting sparse-matrix and correlation operations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:b2a8fb1d5eed0c2c13bc20e3628ea5131d9ce333ac02a28081686d3609fa9b7a
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
  - Python
  - Scanpy
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-confidence-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ranks genes by their positive or negative association with annotation trainability: after Annotatability computes per-cell confidence scores, each gene's expression (scaled to variance 1) is correlated against the per-cell confidence vector to identify disease-state-associated genes.

## When to use

Use when you have an AnnData object with per-cell confidence scores from DNN training dynamics (e.g., mSTZ β cells annotated by condition) and want to rank genes by annotation-trainability association.

## When NOT to use

- The AnnData lacks per-cell confidence scores — run follow_training_dyn_neural_net and probability_list_to_confidence_and_var first, since ranking correlates expression with the confidence vector
- Input is aggregated/convolved spatial transcriptomics data — Annotatability does not directly handle convolved or aggregated data
- The goal is differential expression between arbitrary external labels rather than association with annotation trainability (per-cell confidence from DNN training dynamics)

## Inputs

- AnnData (h5ad) single-cell expression matrix with a label/annotation column (e.g., experimental condition for mSTZ β cells)
- Per-cell confidence scores stored in adata.obs['conf'] (and optionally variability in adata.obs['var']) from the upstream training-dynamics step
- Scaled gene expression matrix (each gene scaled to variance 1, L2 normalization)

## Outputs

- Ranked AnnData (adata_ranked) with per-gene scores in var['conf_score_high'] (positive association) and var['conf_score_low'] (negative association)
- CSV table of ranked genes (e.g., mstz_gene_trainability_scores.csv) with gene identifiers and both score columns

## How to apply

Start from the AnnData object that carries per-cell confidence scores produced by models.follow_training_dyn_neural_net and models.probability_list_to_confidence_and_var (stored in adata.obs['conf']). As a preprocessing step, scale the expression level of each gene to variance 1 (L2 normalization). Then call metrics.rank_genes_conf_min_counts(adata), which ranks genes by the correlation of their expression with the cell confidence vector (A^T·f(û)) and its complement form. Read the per-gene results from adata_ranked.var['conf_score_high'] (positive annotation-trainability association) and adata_ranked.var['conf_score_low'] (negative association), and export the ranked table (e.g., mstz_gene_trainability_scores.csv) with gene identifiers and both score columns. Genes associated with a disease state — such as individual mSTZ β cells — are those whose scaled expression is positively or negatively correlated with the confidence assigned to each cell's disease annotation.

## Related tools

- **Annotatability** (Provides metrics.rank_genes_conf_min_counts, the gene-ranking function that computes conf_score_high/conf_score_low by correlating scaled gene expression with the per-cell confidence vector) — https://github.com/nitzanlab/Annotatability
- **Scanpy** (Base framework for handling the AnnData object and single-cell preprocessing/scaling) — https://scanpy.readthedocs.io/en/stable/
- **pandas** (Dependency used to store/export the ranked gene table (e.g., mstz_gene_trainability_scores.csv))
- **numpy** (Numerical dependency of Annotatability used in the correlation computations)
- **scipy** (Dependency of Annotatability supporting sparse-matrix and correlation operations)

## Examples

```
from Annotatability import metrics; adata_ranked = metrics.rank_genes_conf_min_counts(adata); adata_ranked.var['conf_score_high'], adata_ranked.var['conf_score_low']
```

## Evaluation signals

- adata_ranked.var contains both 'conf_score_high' and 'conf_score_low' columns after the ranking call
- Gene expression was scaled to variance 1 (L2 normalization) before scoring; scores are correlation-like with A^T·f(û) form
- Output CSV contains gene identifiers plus both positive and negative association score columns
- Top positively scored genes should be plausibly associated with the disease state (e.g., mSTZ β-cell disease annotation), consistent with the paper's interpretation
- Pipeline runs end-to-end after confidence scores exist in adata.obs['conf'] from the training-dynamics module

## Limitations

- Genes are scored by correlation with a derived confidence vector, so hyperparameters of upstream training (e.g., training time/epoch_num) influence the scores and are not yet standardized
- Annotatability does not directly handle convolved or aggregated data, such as that obtained from certain spatial transcriptomics technologies
- Annotatability currently treats input annotations as nominal categories only; association scoring inherits this nominal-label assumption

## Evidence

- [full_text] genes associated with a disease state ... are those whose expression levels are positively or negatively correlated with the confidence assigned to each cell's disease annotation, with gene expression scaled to variance 1 beforehand: [evidence span withheld — anchor status: not_found]
