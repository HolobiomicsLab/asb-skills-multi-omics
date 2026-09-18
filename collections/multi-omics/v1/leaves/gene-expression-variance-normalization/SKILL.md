---
name: gene-expression-variance-normalization
description: Use when ranking genes by their positive or negative association with
  per-cell annotation confidence (e.g. Annotatability rank_genes_conf_min_counts on
  mSTZ β-cell data) — variance scaling must precede correlation of expression with
  the confidence vector.
when_to_use_negative:
- When the goal is generic expression normalization for clustering or visualization
  (e.g. Scanpy total-count normalize + log1p) rather than preparing expression for
  annotation-trainability gene-confidence correlation scoring
- When input is convolved or aggregated spatial transcriptomics data, which Annotatability
  does not directly handle
- When per-cell confidence/variability training-dynamics scores have not yet been
  computed — variance scaling alone does not produce gene rankings
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Annotatability
  role: Provides the gene-ranking function metrics.rank_genes_conf_min_counts whose
    preprocessing specification requires scaling each gene's expression to variance
    1 (L2 normalization)
  repo: https://github.com/nitzanlab/Annotatability
- name: Scanpy
  role: Base single-cell framework holding the AnnData expression matrix that is variance-normalized
    prior to gene scoring
  repo: https://scanpy.readthedocs.io/en/stable/
- name: numpy
  role: Numeric backend used for the variance-1 (L2) scaling of gene expression vectors
- name: pandas
  role: Exports the ranked per-gene confidence-score table (e.g. mstz_gene_trainability_scores.csv)
    with both score columns
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:d67f8dfe40fb250f526719921a2fddb049b99aa05fff2b5fb5e334797b25292f
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.2222222222222222
n_claims: 9
---

# gene-expression-variance-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scales the expression level of each gene to variance 1 (L2 normalization) as a preprocessing step before computing annotation-trainability gene association scores, so that gene-confidence correlations are comparable across genes with different expression magnitudes.

## When to use

Use when ranking genes by their positive or negative association with per-cell annotation confidence (e.g. Annotatability rank_genes_conf_min_counts on mSTZ β-cell data) — variance scaling must precede correlation of expression with the confidence vector.

## When NOT to use

- When the goal is generic expression normalization for clustering or visualization (e.g. Scanpy total-count normalize + log1p) rather than preparing expression for annotation-trainability gene-confidence correlation scoring
- When input is convolved or aggregated spatial transcriptomics data, which Annotatability does not directly handle
- When per-cell confidence/variability training-dynamics scores have not yet been computed — variance scaling alone does not produce gene rankings

## Inputs

- AnnData object (Scanpy .h5ad) with gene expression matrix and per-cell confidence scores in adata.obs (e.g. 'conf' column from the training-dynamics step)
- mSTZ β-cell AnnData annotated by experimental condition, carrying confidence scores from follow_training_dyn_neural_net / probability_list_to_confidence_and_var

## Outputs

- Variance-normalized (variance-1 / L2-normalized) gene expression matrix inside the AnnData object
- Ranked gene table (e.g. mstz_gene_trainability_scores.csv) with per-gene positive association scores adata_ranked.var['conf_score_high'] and negative association scores adata_ranked.var['conf_score_low']

## How to apply

After per-cell confidence scores have been computed from DNN training dynamics (via follow_training_dyn_neural_net and probability_list_to_confidence_and_var) and stored in adata.obs, apply variance-1 (L2) normalization to each gene's expression profile across cells in the AnnData object. This preprocessing is explicitly specified for the gene-scoring module so that genes are ranked by correlation/anticorrelation of their (equal-variance) expression with the cell confidence vector A^T·f(û), rather than by expression magnitude. Once scaled, run the Annotatability gene-ranking function metrics.rank_genes_conf_min_counts(adata) and read the per-gene positive and negative association scores from adata_ranked.var['conf_score_high'] and adata_ranked.var['conf_score_low']. Verify that each gene's variance across cells equals 1 before scoring, since unscaled genes with high absolute expression would otherwise dominate the ranking.

## Related tools

- **Annotatability** (Provides the gene-ranking function metrics.rank_genes_conf_min_counts whose preprocessing specification requires scaling each gene's expression to variance 1 (L2 normalization)) — https://github.com/nitzanlab/Annotatability
- **Scanpy** (Base single-cell framework holding the AnnData expression matrix that is variance-normalized prior to gene scoring) — https://scanpy.readthedocs.io/en/stable/
- **numpy** (Numeric backend used for the variance-1 (L2) scaling of gene expression vectors)
- **pandas** (Exports the ranked per-gene confidence-score table (e.g. mstz_gene_trainability_scores.csv) with both score columns)

## Examples

```
from Annotatability import metrics; adata_ranked = metrics.rank_genes_conf_min_counts(adata); adata_ranked.var['conf_score_high'], adata_ranked.var['conf_score_low']
```

## Evaluation signals

- Each gene's expression vector has variance equal to 1 across cells after scaling (L2 normalization check)
- Ranked output AnnData contains the expected per-gene score columns adata_ranked.var['conf_score_high'] and adata_ranked.var['conf_score_low']
- Positive and negative score columns are interpretable as correlation and anticorrelation with the cell confidence vector (A^T·f(û) and its complement form)
- Exported gene table (mstz_gene_trainability_scores.csv) contains gene identifiers plus both score columns, with one row per gene

## Limitations

- Variance scaling equalizes gene variance, so genes expressed in very few cells can produce noisy confidence-association scores; the article's upstream filter retained only genes expressed in more than 3 cells and highly variable (3,000 genes)
- Thresholding and hyperparameter choices (e.g. training time affecting the confidence scores fed into the correlation) are dataset-dependent and not standardized, which propagates into the gene scores

## Evidence

- [results] genes associated with a disease state ... are those whose expression levels are positively or negatively correlated with the confidence assigned to each cell's disease annotation, with gene expression scaled to variance 1 beforehand.: [evidence span withheld — anchor status: not_found]
