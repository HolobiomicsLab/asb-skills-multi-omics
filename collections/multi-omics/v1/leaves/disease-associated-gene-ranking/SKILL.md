---
name: disease-associated-gene-ranking
description: Use when you have single-cell data (e.g. an AnnData object such as mSTZ
  β cells) already annotated and carrying per-cell confidence scores, and you want
  to identify genes whose expression is positively or negatively correlated with a
  disease/annotation signal.
when_to_use_negative:
- Input data is convolved or aggregated (e.g. from certain spatial transcriptomics
  technologies) — Annotatability does not directly handle such data
- Per-cell confidence/variability scores have not yet been computed (i.e. no follow_training_dyn_neural_net
  + probability_list_to_confidence_and_var step was run), so there is no confidence
  vector to correlate gene expression against
- The goal is standard differential expression between label groups rather than ranking
  genes by association with annotation trainability
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Annotatability
  role: Provides metrics.rank_genes_conf_min_counts, the gene-ranking function that
    computes conf_score_high/conf_score_low gene scores
  repo: https://github.com/nitzanlab/Annotatability
- name: models (Annotatability module)
  role: Produces the per-cell confidence vector (follow_training_dyn_neural_net, probability_list_to_confidence_and_var)
    that genes are correlated against
  repo: https://github.com/nitzanlab/Annotatability_notebooks
- name: Scanpy
  role: Base framework for AnnData handling and preprocessing (gene scaling to variance
    1) on which Annotatability is built
  repo: https://scanpy.readthedocs.io/en/stable/
- name: pandas
  role: Exporting the ranked gene table (e.g. mstz_gene_trainability_scores.csv) with
    gene identifiers and both score columns
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:71d8b8b3e7833b80cbb9bdaee73d15b5702d98a678f74b36ea37b505e12ebd7f
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
  - models (Annotatability module)
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# disease-associated-gene-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ranks genes by their positive or negative association with annotation trainability, i.e. by correlation of per-gene expression with the per-cell confidence score produced by deep neural network training dynamics, to surface disease-associated genes (e.g. in mSTZ β cells).

## When to use

Use when you have single-cell data (e.g. an AnnData object such as mSTZ β cells) already annotated and carrying per-cell confidence scores, and you want to identify genes whose expression is positively or negatively correlated with a disease/annotation signal.

## When NOT to use

- Input data is convolved or aggregated (e.g. from certain spatial transcriptomics technologies) — Annotatability does not directly handle such data
- Per-cell confidence/variability scores have not yet been computed (i.e. no follow_training_dyn_neural_net + probability_list_to_confidence_and_var step was run), so there is no confidence vector to correlate gene expression against
- The goal is standard differential expression between label groups rather than ranking genes by association with annotation trainability

## Inputs

- AnnData (adata) single-cell expression matrix annotated with a label key (e.g. mSTZ β cells annotated by experimental condition)
- Per-cell confidence scores stored in adata.obs['conf'] (from follow_training_dyn_neural_net + probability_list_to_confidence_and_var)
- Per-cell variability scores stored in adata.obs['var']

## Outputs

- Ranked AnnData object (adata_ranked) with per-gene scores in adata_ranked.var['conf_score_high'] and adata_ranked.var['conf_score_low']
- CSV table of ranked genes (e.g. mstz_gene_trainability_scores.csv) with gene identifiers and both score columns

## How to apply

Start from the annotated AnnData object (e.g. mSTZ β cells annotated by experimental condition) that carries per-cell confidence scores from the upstream training-dynamics step (follow_training_dyn_neural_net followed by probability_list_to_confidence_and_var). As a mandatory preprocessing step, scale the expression level of each gene to variance 1 (L2 normalization) so that per-gene correlation magnitudes are comparable. Then run the Annotatability gene-ranking function metrics.rank_genes_conf_min_counts(adata), which scores genes by the correlation of their expression with the cell confidence vector (A^T·f(û)) and its complement form. Extract the per-gene scores stored as adata_ranked.var['conf_score_high'] (positive annotation-trainability association) and adata_ranked.var['conf_score_low'] (negative association). The rationale: genes in cells whose disease annotation is easy for the DNN to learn (high confidence) covary with the confidence signal, marking them as disease-state associated; the L2 normalization is explicitly prescribed before computing the association scores.

## Related tools

- **Annotatability** (Provides metrics.rank_genes_conf_min_counts, the gene-ranking function that computes conf_score_high/conf_score_low gene scores) — https://github.com/nitzanlab/Annotatability
- **models (Annotatability module)** (Produces the per-cell confidence vector (follow_training_dyn_neural_net, probability_list_to_confidence_and_var) that genes are correlated against) — https://github.com/nitzanlab/Annotatability_notebooks
- **Scanpy** (Base framework for AnnData handling and preprocessing (gene scaling to variance 1) on which Annotatability is built) — https://scanpy.readthedocs.io/en/stable/
- **pandas** (Exporting the ranked gene table (e.g. mstz_gene_trainability_scores.csv) with gene identifiers and both score columns)

## Examples

```
adata_ranked = metrics.rank_genes_conf_min_counts(adata); adata_ranked.var['conf_score_high']; adata_ranked.var['conf_score_low']
```

## Evaluation signals

- adata_ranked contains var columns 'conf_score_high' and 'conf_score_low' for every gene
- Gene expression was scaled to variance 1 (L2-normalized per gene) before scoring; verify via per-gene variance ≈ 1 in the scored matrix
- Gene scores are consistent with the A^T·f(û) correlation structure: positively correlated genes have high conf_score_high, anticorrelated genes have high conf_score_low
- Ranked output persists as a gene table (e.g. mstz_gene_trainability_scores.csv) with gene identifiers and both score columns
- Known disease-associated genes for the condition (e.g. mSTZ β-cell disease state) appear among top-ranked genes in the expected score direction

## Limitations

- Thresholding on confidence/variability scores for categorizing cells introduces arbitrary cutoffs that could influence downstream gene-score interpretation
- Hyperparameters such as training time (epoch_num) are not standardized and may influence confidence scores, and thus the gene rankings
- Annotatability does not directly handle convolved or aggregated data, such as that obtained from certain spatial transcriptomics technologies

## Evidence

- [discussion] Rank genes by annotation trainability association: [evidence span withheld — anchor status: not_found]
- [results] Gene scoring finding from source card: [evidence span withheld — anchor status: not_found]
