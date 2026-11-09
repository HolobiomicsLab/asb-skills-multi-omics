---
name: gene-annotation-trainability-scoring
description: Use when you have single-cell AnnData with per-cell confidence scores
  from DNN training-dynamics analysis and want to rank genes by their positive or
  negative association with annotation trainability (e.g., disease-related genes in
  mSTZ β cells).
when_to_use_negative:
- Input AnnData lacks per-cell confidence/variability scores — run the upstream training-dynamics
  steps (follow_training_dyn_neural_net, probability_list_to_confidence_and_var) first
- Data are convolved or aggregated, such as counts from certain spatial transcriptomics
  technologies, which Annotatability does not directly handle
- The goal is cell-level classification (easy/hard/ambiguous) or reannotation rather
  than per-gene ranking — use the classification or reannotation modules instead
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Annotatability
  role: Provides metrics.rank_genes_conf_min_counts, the gene-ranking function computing
    annotation-trainability association scores
  repo: https://github.com/nitzanlab/Annotatability
- name: Scanpy
  role: Base framework for AnnData handling and preprocessing (gene variance scaling)
    in the Annotatability code
  repo: https://scanpy.readthedocs.io/en/stable/
- name: numpy
  role: Array computation underlying the correlation of gene expression with the confidence
    vector
- name: pandas
  role: Export of the ranked gene score table (e.g., mstz_gene_trainability_scores.csv)
- name: scipy
  role: Dependency for sparse/graph and statistical operations in Annotatability's
    metrics module
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:87c787769a242ee38608fbd3517439ddd58421042bcc609bb964592b493bed13
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# gene-annotation-trainability-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotatability gene-ranking skill that scores each gene by the correlation of its variance-normalized expression with the per-cell confidence score, yielding positive (conf_score_high) and negative (conf_score_low) annotation-trainability association scores that identify disease- or state-related genes.

## When to use

Use when you have single-cell AnnData with per-cell confidence scores from DNN training-dynamics analysis and want to rank genes by their positive or negative association with annotation trainability (e.g., disease-related genes in mSTZ β cells).

## When NOT to use

- Input AnnData lacks per-cell confidence/variability scores — run the upstream training-dynamics steps (follow_training_dyn_neural_net, probability_list_to_confidence_and_var) first
- Data are convolved or aggregated, such as counts from certain spatial transcriptomics technologies, which Annotatability does not directly handle
- The goal is cell-level classification (easy/hard/ambiguous) or reannotation rather than per-gene ranking — use the classification or reannotation modules instead

## Inputs

- AnnData object (e.g., .h5ad) of single-cell expression with cell annotations (e.g., mSTZ β cells by experimental condition)
- Per-cell confidence scores stored in adata.obs['conf'] (from follow_training_dyn_neural_net and probability_list_to_confidence_and_var)
- Gene expression matrix to be scaled to variance 1 per gene before scoring

## Outputs

- adata_ranked.var['conf_score_high']: per-gene positive annotation-trainability association score
- adata_ranked.var['conf_score_low']: per-gene negative annotation-trainability association score
- Ranked gene table (e.g., mstz_gene_trainability_scores.csv) with gene identifiers and both score columns

## How to apply

Start from the AnnData object (e.g., mSTZ β cells annotated by experimental condition) that already carries per-cell confidence values from the upstream steps follow_training_dyn_neural_net and probability_list_to_confidence_and_var. As preprocessing, scale the expression level of each gene to variance 1 (L2 normalization), as specified for the gene-scoring module. Then run the Annotatability ranking function metrics.rank_genes_conf_min_counts(adata), which ranks genes by correlation of their expression with the cell confidence vector (A^T·f(û)) and its complement form, capturing both positive and negative association with annotation trainability. Extract per-gene scores from adata_ranked.var['conf_score_high'] (positive association) and adata_ranked.var['conf_score_low'] (negative association) and export the ranked table, e.g., as mstz_gene_trainability_scores.csv with gene identifiers and both score columns.

## Related tools

- **Annotatability** (Provides metrics.rank_genes_conf_min_counts, the gene-ranking function computing annotation-trainability association scores) — https://github.com/nitzanlab/Annotatability
- **Scanpy** (Base framework for AnnData handling and preprocessing (gene variance scaling) in the Annotatability code) — https://scanpy.readthedocs.io/en/stable/
- **numpy** (Array computation underlying the correlation of gene expression with the confidence vector)
- **pandas** (Export of the ranked gene score table (e.g., mstz_gene_trainability_scores.csv))
- **scipy** (Dependency for sparse/graph and statistical operations in Annotatability's metrics module)

## Examples

```
adata_ranked = metrics.rank_genes_conf_min_counts(adata); adata_ranked.var['conf_score_high']; adata_ranked.var['conf_score_low']
```

## Evaluation signals

- adata_ranked.var contains both 'conf_score_high' and 'conf_score_low' columns for every gene
- Every gene's expression was scaled to variance 1 prior to scoring; verify per-gene variance ≈ 1 in the preprocessed matrix
- Genes known to mark the disease state (e.g., mSTZ β-cell condition) appear among high |conf_score| genes with consistent sign
- The ranked table (e.g., mstz_gene_trainability_scores.csv) contains gene identifiers plus both score columns with no missing values
- Scores are interpretable as correlation/anticorrelation with the confidence vector (A^T·f(û)), so values should fall within correlation-comparable ranges

## Limitations

- Annotatability treats input annotations as nominal categories only; probabilistic and continuous annotations are deferred to future work
- Confidence/variability scores (and hence gene ranks) depend on hyperparameters such as training time (epoch_num), which are not yet standardized
- The percentile threshold q for hard-to-learn categorization is chosen dataset-dependently (25%–90%), introducing arbitrary cutoffs that can influence downstream interpretation
- Annotatability does not directly handle convolved or aggregated data, such as that obtained from certain spatial transcriptomics technologies

## Evidence

- [discussion] confidence definition grounding: [evidence span withheld — anchor status: not_found]
- [results] gene association semantics: [evidence span withheld — anchor status: not_found]
