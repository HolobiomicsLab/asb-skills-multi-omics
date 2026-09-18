---
name: fdr-tpr-operating-point-analysis
description: 'Use when evaluating a variable selection procedure that promises false
  discovery rate control: you need empirical FDR vs. TPR operating points across target
  FDR levels (e.g., 0.05, 0.1, 0.2) and across importance score functions of differing
  predictive adequacy.'
when_to_use_negative:
- Input AnnData contains normalized expression rather than raw counts — VIVS requires
  raw counts and FDR calibration may be invalid otherwise
- You need a single calibrated discovery call rather than a benchmark sweep of operating
  points across importance scores and FDR levels
- The feature set is unfiltered with many thousands of genes — VIVS may not scale;
  use select_genes first or choose a different approach
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: VI-VS / VIVS
  role: Provides the CRT-based variable selection framework whose FDR calibration
    and power are benchmarked; trained via VIVS(...).train_all() to fit the VAE generative
    model
  repo: github.com/YosefLab/VIVS
- name: Jax
  role: Backend for the fast, JIT-compiled VIVS inference used to generate Monte Carlo
    synthetic samples in the CRT
- name: scanpy
  role: Used to import the input data as an AnnData object with raw counts and responses
    in obsm
- name: scVI
  role: Example generative model providing the normalized gene expression of the VAE
    decoder used to draw synthetic features for the CRT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:f4c9739206e1164408294994782f52ea3db4f51f1f94ca99ae7deda3ff43f826
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# fdr-tpr-operating-point-analysis

## Summary

Benchmarking protocol for VI-VS variable selection that sweeps importance score choices (linear OLS vs. MLP predictors of varying capacity) and target FDR levels, recording empirical FDR/TPR operating points and held-out MSE for each score in fig3bc_fdr_tpr_table.csv.

## When to use

Use when evaluating a variable selection procedure that promises false discovery rate control: you need empirical FDR vs. TPR operating points across target FDR levels (e.g., 0.05, 0.1, 0.2) and across importance score functions of differing predictive adequacy.

## When NOT to use

- Input AnnData contains normalized expression rather than raw counts — VIVS requires raw counts and FDR calibration may be invalid otherwise
- You need a single calibrated discovery call rather than a benchmark sweep of operating points across importance scores and FDR levels
- The feature set is unfiltered with many thousands of genes — VIVS may not scale; use select_genes first or choose a different approach

## Inputs

- Semi-synthetic Gaussian-noise single-cell dataset as a scanpy AnnData object with raw counts (not normalized)
- Response variable(s) Y stored in the AnnData obsm attribute (referenced via feature_obsm_key)
- Trained VIVS model (VAE generative model fit independent of the response)
- Importance score configurations: linear OLS and MLPs of increasing capacity (e.g., n_hidden=8 and larger)

## Outputs

- fig3bc_fdr_tpr_table.csv with columns: importance_score, target_FDR, empirical_FDR, TPR, held-out MSE
- Conditional randomization test p-values per feature for each importance score

## How to apply

Load the semi-synthetic Gaussian-noise dataset as a scanpy AnnData object with raw counts (never normalized data), filter genes with select_genes (e.g., n_top_genes=2000) since VIVS may not scale to many thousands of genes, then train the VIVS model with model.train_all() so the VAE generative model captures feature dependencies independent of the response. Fit multiple importance score models on held-out data: a linear OLS regression (xy_linear=True equivalent) and MLP predictors of increasing capacity (e.g., xy_model_kwargs={'n_hidden': 8} and larger). For each score, compute conditional randomization test (CRT) p-values per feature by drawing K Monte Carlo synthetic samples per gene from the VAE decoder and comparing T(X_tilde, Y, S) to T(X, Y, S). Apply the Benjamini-Hochberg procedure at target FDR levels 0.05, 0.1, and 0.2 and record empirical FDR and TPR per score; also compute each score's held-out MSE to assess predictive adequacy. Write rows (importance_score, target_FDR, empirical_FDR, TPR, held-out MSE) to fig3bc_fdr_tpr_table.csv and interpret: lower held-out MSE scores should yield higher TPR at controlled FDR.

## Related tools

- **VI-VS / VIVS** (Provides the CRT-based variable selection framework whose FDR calibration and power are benchmarked; trained via VIVS(...).train_all() to fit the VAE generative model) — github.com/YosefLab/VIVS
- **Jax** (Backend for the fast, JIT-compiled VIVS inference used to generate Monte Carlo synthetic samples in the CRT)
- **scanpy** (Used to import the input data as an AnnData object with raw counts and responses in obsm)
- **scVI** (Example generative model providing the normalized gene expression of the VAE decoder used to draw synthetic features for the CRT)

## Examples

```
from vivs import VIVS; model = VIVS(adata, feature_obsm_key='my_obsm_key', xy_linear=False, xy_model_kwargs={'n_hidden': 8}); model.train_all()
```

## Evaluation signals

- fig3bc_fdr_tpr_table.csv exists and contains all five required columns: importance_score, target_FDR, empirical_FDR, TPR, held-out MSE
- Empirically FDR-controlled runs show empirical FDR at or below the target level (0.05, 0.1, 0.2) for each importance score
- Importance scores with lower held-out MSE (more adequate predictors) yield higher TPR at the same target FDR, matching the paper's stated expectation
- Even a deliberately weak predictor (linear OLS) still controls FDR, so its operating point should show low empirical FDR despite poor power
- p-values are produced by a genuine conditional randomization test (K Monte Carlo samples per gene from the VAE decoder) followed by Benjamini-Hochberg

## Limitations

- VIVS may not scale to many thousands of genes; genes should be filtered beforehand (e.g., select_genes with n_top_genes=2000)
- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries
- Right now, VIVS only supports assays where X corresponds to gene expression counts; applicability to sparse ATAC-seq data may require a generative model accounting for accessibility sparsity

## Evidence

- [intro] We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate.: "We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate."
- [methods] import the data of intest in a scanpy AnnData object.: "import the data of intest in a scanpy AnnData object."
- [readme] In particular, make sure than the anndata contains raw counts and not normalized data.: "In particular, make sure than the anndata contains raw counts and not normalized data."
- [readme] VIVS may not scale to many thousands of genes.: "VIVS may not scale to many thousands of genes."
