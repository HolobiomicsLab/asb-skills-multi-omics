---
name: held-out-prediction-error-evaluation
description: Use when running VI-VS/CRT variable selection and you must choose an
  importance score function (e.g., xy_linear OLS vs. MLP with n_hidden capacity),
  or when comparing FDR-TPR trade-offs across importance score choices in semi-synthetic
  experiments.
when_to_use_negative:
- The anndata contains normalized expression instead of raw counts — VIVS only supports
  assays where X corresponds to gene expression counts, so held-out MSE comparisons
  would be run on invalid input
- The dataset has many thousands of genes without prior filtering — VIVS may not scale;
  genes should first be reduced (e.g., select_genes with n_top_genes=2000) before
  fitting and scoring importance models
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: VI-VS
  role: 'Framework being evaluated: trains a VAE generative model and computes conditional
    randomization test p-values using candidate importance scores'
  repo: https://github.com/YosefLab/VIVS
- name: Python
  role: Language in which VI-VS is implemented and in which the held-out MSE and FDR-TPR
    evaluation scripts are run
  repo: https://github.com/YosefLab/VIVS
- name: Jax
  role: Backend providing parallel computing and just-in-time compilation for fast
    VIVS inference when fitting multiple importance-score candidates
  repo: https://github.com/YosefLab/VIVS
- name: scanpy
  role: Used to import the data of interest as a scanpy AnnData object with raw counts
    before model training and evaluation
- name: scVI
  role: Generative model family for normalized gene expression whose decoder supplies
    the Monte Carlo synthetic samples used in the CRT statistic
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:4ef29cb63dc1c29dec5bb451146588f456b13a656e632ffaa42e93ecd085f411
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VIVS
  tools:
  - Jax
  - Python
  - VI-VS
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# held-out-prediction-error-evaluation

## Summary

Fits candidate importance-score models (linear OLS vs. increasing-capacity MLPs) for VI-VS on held-out data and compares their held-out MSE to select an importance score for conditional randomization tests. The paper's expectation is that more adequate predictors (lower held-out MSE) yield more powerful, FDR-controlled VI-VS tests.

## When to use

Use when running VI-VS/CRT variable selection and you must choose an importance score function (e.g., xy_linear OLS vs. MLP with n_hidden capacity), or when comparing FDR-TPR trade-offs across importance score choices in semi-synthetic experiments.

## When NOT to use

- The anndata contains normalized expression instead of raw counts — VIVS only supports assays where X corresponds to gene expression counts, so held-out MSE comparisons would be run on invalid input
- The dataset has many thousands of genes without prior filtering — VIVS may not scale; genes should first be reduced (e.g., select_genes with n_top_genes=2000) before fitting and scoring importance models

## Inputs

- Gaussian-noise semi-synthetic single-cell dataset as a scanpy AnnData object with raw counts (X = gene expression; response Y in obsm)
- VIVS model trained on the raw-count AnnData (model.train_all())
- Candidate importance-score model specifications: linear OLS (xy_linear=True) and MLPs of varying capacity (e.g., xy_model_kwargs={'n_hidden': 8} and larger)

## Outputs

- Per-feature conditional randomization test p-values for each importance score
- Benjamini-Hochberg calls at target FDR levels 0.05, 0.1, and 0.2 per importance score
- fig3bc_fdr_tpr_table.csv with rows of (importance_score, target_FDR, empirical_FDR, TPR, held-out MSE) for FDR-TPR curves

## How to apply

Load the single-cell dataset as a scanpy AnnData object containing raw counts (not normalized data), optionally filter genes with select_genes (e.g., n_top_genes=2000), then initialize and train the VIVS generative model with model.train_all() so the VAE decoder captures feature dependencies independent of the response. Fit importance-score models on held-out data: a linear OLS regression (xy_linear=True equivalent) and MLP predictors of increasing capacity such as xy_model_kwargs={'n_hidden': 8} and larger. For each candidate importance score, compute per-feature conditional randomization test p-values by drawing K Monte Carlo synthetic samples per gene from the VAE decoder and comparing T(X_tilde, Y, S) to T(X, Y, S). Apply Benjamini-Hochberg at target FDR levels 0.05, 0.1, and 0.2, record empirical FDR and TPR operating points, and compute each score's held-out MSE. Judge adequacy by the paper's stated expectation: FDR should remain controlled (empirical FDR at or below target even for the linear-OLS score), and models with lower held-out predictive MSE should produce more powerful tests (higher TPR), as observed when higher-capacity MLP scores increased power in the Gaussian-noise semi-synthetic experiment.

## Related tools

- **VI-VS** (Framework being evaluated: trains a VAE generative model and computes conditional randomization test p-values using candidate importance scores) — https://github.com/YosefLab/VIVS
- **Python** (Language in which VI-VS is implemented and in which the held-out MSE and FDR-TPR evaluation scripts are run) — https://github.com/YosefLab/VIVS
- **Jax** (Backend providing parallel computing and just-in-time compilation for fast VIVS inference when fitting multiple importance-score candidates) — https://github.com/YosefLab/VIVS
- **scanpy** (Used to import the data of interest as a scanpy AnnData object with raw counts before model training and evaluation)
- **scVI** (Generative model family for normalized gene expression whose decoder supplies the Monte Carlo synthetic samples used in the CRT statistic)

## Examples

```
model = VIVS(adata, feature_obsm_key='my_obsm_key', xy_linear=False, xy_model_kwargs={'n_hidden': 8}); model.train_all()
```

## Evaluation signals

- fig3bc_fdr_tpr_table.csv exists with all five columns: importance_score, target_FDR, empirical_FDR, TPR, held-out MSE
- FDR-controlled runs show empirical FDR at or below target levels (0.05, 0.1, 0.2) — even the linear-OLS importance score kept FDR control despite being a poor predictor
- Held-out MSE and TPR are inversely related: models with the best held-out predictive performance (lower MSE) detected more true positives (higher TPR)
- Higher-capacity MLP scores (e.g., n_hidden=8 and larger) show increased power relative to the linear score at matched target FDR
- Per-feature CRT p-values are valid: computed by comparing T(X_tilde, Y, S) against T(X, Y, S) with K Monte Carlo draws per gene from the VAE decoder

## Limitations

- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries regardless of importance-score adequacy
- A lower held-out MSE guides selection empirically but the linear-OLS score still controlled FDR despite poor prediction, so held-out performance primarily affects power, not FDR calibration
- VIVS may not scale to many thousands of genes; genes must be filtered beforehand (e.g., select_genes, n_top_genes=2000)

## Evidence

- [readme] VIVS may not scale to many thousands of genes. In such a case, it is recommended to filter the genes before running VIVS: "VIVS may not scale to many thousands of genes. In such a case, it is recommended to filter the genes before running VIVS, which can be done in the following way:"
- [readme] Raw counts requirement for AnnData input: "In particular, make sure than the anndata contains raw counts and not normalized data."
- [intro] CRT with calibrated FDR estimation: [evidence span withheld — anchor status: not_found]
- [methods] Data import as scanpy AnnData object: "import the data of intest in a scanpy AnnData object."
