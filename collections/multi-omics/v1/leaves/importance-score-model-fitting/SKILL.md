---
name: importance-score-model-fitting
description: 'Use when decision support for choosing the importance score T(X,Y,S)
  in VI-VS: when testing conditional gene dependencies and deciding between a linear
  score (xy_linear=True) or nonlinear MLP scores, or when held-out predictive performance
  should be used to justify the score choice.'
when_to_use_negative:
- The AnnData contains normalized expression rather than raw counts — VIVS only supports
  assays where X corresponds to gene expression counts
- The dataset has many thousands of genes without prior filtering — VIVS may not scale;
  use select_genes(adata, n_top_genes=2000) first instead of fitting scores on the
  full feature set
- The goal is marginal association (e.g. plain co-expression networks) rather than
  calibrated conditional dependency testing with FDR control
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: VI-VS / VIVS
  role: Framework whose trained VIVS model provides the VAE decoder used to draw synthetic
    samples for the conditional randomization test with the fitted importance scores
  repo: https://github.com/YosefLab/VIVS
- name: Jax
  role: Backend providing parallel computing and just-in-time compilation to speed
    up VIVS inference and Monte Carlo sampling
- name: scanpy
  role: Used to import the data of interest as an AnnData object with raw counts as
    VIVS input
- name: scVI
  role: Generative model for normalized gene expression underlying the VAE decoder
    used for CRT synthetic samples
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:b981298df1433bd0a9179cb9b614830472384dd394ce143ecad46e511c7f48ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VIVS
  tools:
  - Jax
  - Python
  - VI-VS
  - VI-VS / VIVS
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# importance-score-model-fitting

## Summary

Fits and compares importance-score functions (linear OLS vs. MLP predictors of varying capacity) within the VI-VS conditional randomization test framework, then evaluates each choice via Benjamini-Hochberg-controlled FDR/TPR curves and held-out MSE to guide score selection in single-cell multiomic feature-dependency testing.

## When to use

Decision support for choosing the importance score T(X,Y,S) in VI-VS: when testing conditional gene dependencies and deciding between a linear score (xy_linear=True) or nonlinear MLP scores, or when held-out predictive performance should be used to justify the score choice.

## When NOT to use

- The AnnData contains normalized expression rather than raw counts — VIVS only supports assays where X corresponds to gene expression counts
- The dataset has many thousands of genes without prior filtering — VIVS may not scale; use select_genes(adata, n_top_genes=2000) first instead of fitting scores on the full feature set
- The goal is marginal association (e.g. plain co-expression networks) rather than calibrated conditional dependency testing with FDR control

## Inputs

- scanpy AnnData object with raw gene expression counts (X) and response variables Y in obsm (e.g. feature_obsm_key='my_obsm_key')
- VIVS model trained with model.train_all() on the AnnData (Jax backend, GPU recommended)
- Held-out data split for fitting importance-score models and computing held-out MSE

## Outputs

- Conditional randomization test p-values per feature for each importance score
- Benjamini-Hochberg rejections at target FDR levels 0.05, 0.1, 0.2 with empirical FDR and TPR operating points
- fig3bc_fdr_tpr_table.csv with columns: importance_score, target_FDR, empirical_FDR, TPR, held-out MSE

## How to apply

Load the semi-synthetic or real dataset as a scanpy AnnData object containing raw counts (not normalized data) and train the VIVS model (model.train_all()) so the VAE generative model captures feature dependencies independent of the response Y stored in obsm. Fit multiple importance-score models on held-out data: a linear OLS regression (xy_linear=True equivalent) and MLP predictors of increasing capacity, e.g. xy_model_kwargs={"n_hidden": 8} and larger. For each score, compute conditional randomization test p-values per feature by drawing K Monte Carlo synthetic samples for each gene from the VAE decoder and comparing T(X_tilde, Y, S) to T(X, Y, S). Apply the Benjamini-Hochberg procedure at target FDR levels 0.05, 0.1, and 0.2, recording empirical FDR and TPR operating points, and record each score model's held-out mean squared error. Write all rows (importance_score, target_FDR, empirical_FDR, TPR, held-out MSE) to fig3bc_fdr_tpr_table.csv; the rationale is that more adequate predictors (lower held-out MSE) yield more powerful tests while preserving the CRT's FDR calibration.

## Related tools

- **VI-VS / VIVS** (Framework whose trained VIVS model provides the VAE decoder used to draw synthetic samples for the conditional randomization test with the fitted importance scores) — https://github.com/YosefLab/VIVS
- **Jax** (Backend providing parallel computing and just-in-time compilation to speed up VIVS inference and Monte Carlo sampling)
- **scanpy** (Used to import the data of interest as an AnnData object with raw counts as VIVS input)
- **scVI** (Generative model for normalized gene expression underlying the VAE decoder used for CRT synthetic samples)

## Examples

```
model = VIVS(adata, feature_obsm_key="my_obsm_key", xy_linear=False, xy_model_kwargs={"n_hidden": 8}); model.train_all()
```

## Evaluation signals

- fig3bc_fdr_tpr_table.csv exists with all five columns: importance_score, target_FDR, empirical_FDR, TPR, held-out MSE
- For FDR-controlled runs, empirical FDR is at or below the target levels (0.05, 0.1, 0.2) — the paper theoretically proves calibrated FDR estimation
- Higher-capacity MLP scores (e.g. n_hidden=8 and larger) yield higher TPR than the linear OLS score while still controlling FDR
- Importance-score models with lower held-out MSE (better predictive adequacy) detect more true positives (higher TPR)
- FDR control holds even for the linear OLS score despite it being a poor predictor, confirming calibration is independent of score adequacy

## Limitations

- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries
- VIVS may not scale to many thousands of genes; gene filtering (e.g. n_top_genes=2000) is recommended beforehand
- Even an adequate importance score does not guarantee power if the score is a poor predictor — linear OLS kept FDR control but was a poor predictor with correspondingly lower power

## Evidence

- [discussion] VI-VS can be employed as a meta-algorithm to make the discoveries of existing methods more interpretable by constructing importance scores from their predictions.: "VI-VS can be employed as a meta-algorithm to make the discoveries of existing methods more interpretable by constructing importance scores from their predictions."
- [intro] We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate.: "We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate."
- [readme] model = VIVS(
    adata,
    feature_obsm_key="my_obsm_key",
    xy_linear=False,
    xy_model_kwargs={"n_hidden": 8}
): "xy_linear=False,
    xy_model_kwargs={"n_hidden": 8}
)
model.train_all()"
- [readme] In particular, make sure than the anndata contains raw counts and not normalized data.: "In particular, make sure than the anndata contains raw counts and not normalized data."
- [methods] We implemented VI-VS in a fast and scalable way that is available as an open-source Python package.: "We implemented VI-VS in a fast and scalable way that is available as an open-source Python package."
