---
name: semi-synthetic-single-cell-data-simulation
description: Use when you need ground-truth conditional dependencies to benchmark
  FDR control and power of feature-selection methods on single-cell data, or to compare
  importance-score choices (e.g., linear OLS vs MLP) via held-out predictive performance.
when_to_use_negative:
- When the goal is discovery on real (non-perturbed) data rather than benchmarking
  against known ground-truth dependencies
- When data are already normalized — VIVS requires raw counts in the AnnData object,
  and simulation on normalized values would distort the count distribution
- When the number of features is very large — VIVS may not scale to many thousands
  of genes, so features should first be filtered (e.g., select_genes with n_top_genes=2000)
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: VI-VS
  role: 'Framework under benchmark: its generative model supplies the VAE decoder
    used to draw synthetic samples for the conditional randomization test'
  repo: https://github.com/YosefLab/VIVS
- name: scanpy
  role: Loads and holds the semi-synthetic dataset as a scanpy AnnData object with
    raw counts
- name: Jax
  role: Backend providing parallel computing and just-in-time compilation to speed
    up VIVS inference
- name: scVI
  role: Generative model for normalized gene expression underlying the VIVS decoder
    used in the semi-synthetic evaluation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:8a27869bfbd575a755241e72881577e056cd5e6aa8dc98be2069bdb9fdd76e50
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# semi-synthetic-single-cell-data-simulation

## Summary

Generate a semi-synthetic single-cell benchmark by adding Gaussian noise to real expression data so that ground-truth feature-response dependencies are known, enabling calibration and power (FDR/TPR) evaluation of variable-selection methods such as VI-VS.

## When to use

Use when you need ground-truth conditional dependencies to benchmark FDR control and power of feature-selection methods on single-cell data, or to compare importance-score choices (e.g., linear OLS vs MLP) via held-out predictive performance.

## When NOT to use

- When the goal is discovery on real (non-perturbed) data rather than benchmarking against known ground-truth dependencies
- When data are already normalized — VIVS requires raw counts in the AnnData object, and simulation on normalized values would distort the count distribution
- When the number of features is very large — VIVS may not scale to many thousands of genes, so features should first be filtered (e.g., select_genes with n_top_genes=2000)

## Inputs

- Single-cell expression dataset as a scanpy AnnData object with raw counts (not normalized data)
- Response(s) Y stored in the AnnData obsm attribute (array or dataframe), referenced via feature_obsm_key
- Gaussian noise specification for the semi-synthetic perturbation
- Importance-score model specifications (linear OLS; MLPs with xy_model_kwargs such as n_hidden)

## Outputs

- Conditional randomization test p-values per feature (from K Monte Carlo draws per gene from the VAE decoder)
- FDR/TPR operating points at Benjamini-Hochberg target FDR levels 0.05, 0.1, 0.2 for each importance score
- Held-out MSE for each importance-score model
- Results table (e.g., fig3bc_fdr_tpr_table.csv) with columns: importance_score, target_FDR, empirical_FDR, TPR, held-out MSE

## How to apply

Load real single-cell expression data as a scanpy AnnData object and perturb it with Gaussian noise to create a semi-synthetic dataset with known response-feature relationships, ensuring raw counts rather than normalized data are used. Fit the VIVS generative model with model.train_all() so the VAE decoder captures feature dependencies independently of the response. Fit candidate importance-score models on held-out data — a linear OLS regression and MLP predictors of increasing capacity (e.g., xy_model_kwargs={'n_hidden': 8} and larger) — then compute conditional randomization test p-values per feature by drawing K Monte Carlo synthetic samples per gene from the VAE decoder and comparing T(X_tilde, Y, S) to T(X, Y, S). Apply the Benjamini-Hochberg procedure at target FDR levels 0.05, 0.1, and 0.2 to obtain empirical FDR and TPR operating points per importance score. Record each score's held-out MSE alongside its operating points so held-out predictive performance can guide importance-score selection.

## Related tools

- **VI-VS** (Framework under benchmark: its generative model supplies the VAE decoder used to draw synthetic samples for the conditional randomization test) — https://github.com/YosefLab/VIVS
- **scanpy** (Loads and holds the semi-synthetic dataset as a scanpy AnnData object with raw counts)
- **Jax** (Backend providing parallel computing and just-in-time compilation to speed up VIVS inference)
- **scVI** (Generative model for normalized gene expression underlying the VIVS decoder used in the semi-synthetic evaluation)

## Examples

```
from vivs import VIVS, select_genes; adata = select_genes(adata, n_top_genes=2000); model = VIVS(adata, feature_obsm_key="my_obsm_key", xy_linear=False, xy_model_kwargs={"n_hidden": 8}); model.train_all()
```

## Evaluation signals

- The output table exists with all five columns: importance_score, target_FDR, empirical_FDR, TPR, held-out MSE
- Empirical FDR is at or below the target BH level (0.05, 0.1, or 0.2) for each importance score, confirming FDR control
- More adequate predictors (lower held-out MSE) yield higher TPR, matching the paper's expectation that better-held-out-performance scores are more powerful
- Even the linear OLS importance score, despite being a poor predictor, maintains FDR control — a useful sanity invariant
- The AnnData object passed to VIVS contains raw counts, not normalized values

## Limitations

- VIVS may not scale to many thousands of genes; genes should be filtered first (e.g., select_genes with n_top_genes=2000)
- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries
- No software version information is provided for the generative model or VI-VS implementation, which may hinder exact reproduction

## Evidence

- [readme] Right now, VIVS only supports assays where $X$ corresponds to gene expression counts. In particular, make sure than the anndata contains raw counts and not normalized data.: "Right now, VIVS only supports assays where $X$ corresponds to gene expression counts. In particular, make sure than the anndata contains raw counts and not normalized data."
- [abstract] It uses deep generative models to identify conditionally dependent features, with false discovery rate control.: "It uses deep generative models to identify conditionally dependent features, with false discovery rate control."
- [methods] model = VIVS(
    adata,
    feature_obsm_key="my_obsm_key",
    xy_linear=False,
    xy_model_kwargs={"n_hidden": 8}
): "model = VIVS(
    adata,
    feature_obsm_key="my_obsm_key",
    xy_linear=False,
    xy_model_kwargs={"n_hidden": 8}
)"
- [readme] VIVS may not scale to many thousands of genes. In such a case, it is recommended to filter the genes before running VIVS: "VIVS may not scale to many thousands of genes. In such a case, it is recommended to filter the genes before running VIVS"
