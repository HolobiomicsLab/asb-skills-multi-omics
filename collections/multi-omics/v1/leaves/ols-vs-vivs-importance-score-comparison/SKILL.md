---
name: ols-vs-vivs-importance-score-comparison
description: Use when comparing feature-importance scorers for conditional independence
  testing on count-valued omics data, especially when the response is generated with
  non-Gaussian (Poisson) noise and FDR calibration must be verified.
when_to_use_negative:
- The response variable is a Gaussian-noised linear function of the features, where
  the OLS baseline is expected to remain calibrated and the Poisson stress test adds
  no discriminating information
- The AnnData object contains normalized data instead of raw counts, since VIVS explicitly
  requires raw gene expression counts
- The feature matrix has many thousands of genes beyond VIVS's scaling limit and has
  not been filtered (e.g., via select_genes with n_top_genes=2000)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: VI-VS
  role: Primary method under test; provides the deep generative model, MLP importance
    score, and calibrated CRT p-values with FDR control
  repo: github.com/YosefLab/VIVS
- name: Python
  role: Implementation language; VI-VS is available as an open-source Python package
    requiring Python >= 3.9
  repo: github.com/YosefLab/VIVS
- name: Jax
  role: Backend for parallel computing and just-in-time compilation to speed up VI-VS
    inference
- name: scanpy
  role: 'Data import format: the count matrix and responses are handled as a scanpy
    AnnData object with raw counts; select_genes from vivs filters to top genes'
- name: scVI
  role: Generative model whose normalized gene expression underlies VI-VS feature
    conditioning; an alternative generative model was stress-tested for single-cell
    RNA data
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:3799259eb11222e983ce4e1cedddd35a7f3ec1e60089701cd7683ef99388655c
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# ols-vs-vivs-importance-score-comparison

## Summary

Benchmark that contrasts the realized false discovery rate (FDR) of an OLS-based importance score against VI-VS with an MLP importance score on semi-synthetic single-cell count data (PBMC), including the Poisson-response stress test where OLS loses calibration while VI-VS remains calibrated.

## When to use

Use when comparing feature-importance scorers for conditional independence testing on count-valued omics data, especially when the response is generated with non-Gaussian (Poisson) noise and FDR calibration must be verified.

## When NOT to use

- The response variable is a Gaussian-noised linear function of the features, where the OLS baseline is expected to remain calibrated and the Poisson stress test adds no discriminating information
- The AnnData object contains normalized data instead of raw counts, since VIVS explicitly requires raw gene expression counts
- The feature matrix has many thousands of genes beyond VIVS's scaling limit and has not been filtered (e.g., via select_genes with n_top_genes=2000)

## Inputs

- Semi-synthetic gene expression count matrix with correlated features (PBMC-derived, raw counts in scanpy AnnData format)
- Response variable y generated as Poisson counts conditionally dependent on a designated feature
- Nuisance/technical covariates for the OLS regression baseline
- Number of Monte Carlo randomized copies K per feature for the CRT

## Outputs

- CRT p-values per feature for VI-VS (MLP importance score)
- CRT p-values per feature for the OLS prediction-error importance score
- Comparison table of realized FDR versus target FDR level (Benjamini-Hochberg) for both methods
- Plot of realized FDR against target FDR, mirroring Additional file 1: Figure S1

## How to apply

Simulate a semi-synthetic feature matrix of correlated gene expression counts (PBMC-based) and draw the response y as Poisson counts conditionally dependent on a designated feature, replacing the Gaussian-noised linear response used elsewhere. Fit VI-VS by training its deep generative model (scVI-style, Jax backend) on the count matrix and computing conditional randomization test (CRT) p-values per feature via Monte Carlo with K randomized copies of each gene drawn from a fixed posterior sample z-bar. Fit the OLS baseline by computing importance scores as the prediction error of a linear regression of y on X and nuisance factors, then derive its CRT p-values with the same K randomizations. Apply the Benjamini-Hochberg procedure to both methods' p-values over a grid of target FDR levels and record realized FDR per level, mirroring Additional file 1: Figure S1. Decision rule: a method is calibrated only if its null p-values stochastically dominate the uniform distribution and its realized FDR curve tracks the target level.

## Related tools

- **VI-VS** (Primary method under test; provides the deep generative model, MLP importance score, and calibrated CRT p-values with FDR control) — github.com/YosefLab/VIVS
- **Python** (Implementation language; VI-VS is available as an open-source Python package requiring Python >= 3.9) — github.com/YosefLab/VIVS
- **Jax** (Backend for parallel computing and just-in-time compilation to speed up VI-VS inference)
- **scanpy** (Data import format: the count matrix and responses are handled as a scanpy AnnData object with raw counts; select_genes from vivs filters to top genes)
- **scVI** (Generative model whose normalized gene expression underlies VI-VS feature conditioning; an alternative generative model was stress-tested for single-cell RNA data)

## Examples

```
from vivs import VIVS; model = VIVS(adata, feature_obsm_key='my_obsm_key', xy_linear=False, xy_model_kwargs={'n_hidden': 8}); model.train_all()
```

## Evaluation signals

- Realized FDR curve of VI-VS tracks the target FDR level under Benjamini-Hochberg in the Poisson-response scenario, while the OLS curve is inflated
- Null CRT p-values stochastically dominate the uniform distribution for the calibrated method
- The same K randomized copies and the same BH grid of target FDR levels are applied to both methods so the comparison is apples-to-apples
- VI-VS CRT p-values are computed from randomized copies generated from a fixed posterior sample z-bar, matching the Monte Carlo scheme described in the paper
- Realized-FDR-versus-target-FDR plot reproduces the qualitative pattern of Additional file 1: Figure S1

## Limitations

- VIVS may not scale to many thousands of genes; genes must be pre-filtered (e.g., select_genes with n_top_genes=2000)
- VIVS only supports assays where X corresponds to gene expression counts and requires raw counts, not normalized data
- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries
- No software version information is provided for the generative model or VI-VS implementation, hindering exact reproduction

## Evidence

- [results] When the semi-synthetic response variable counts were generated from a Poisson distribution, OLS performed worse, whereas VI-VS with an MLP importance score controlled the FDR in both simulation scenarios.: [evidence span withheld — anchor status: not_found]
- [intro] Calibrated FDR estimation claim: "We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate."
- [methods] Jax backend for inference: "Our implementation first relies on parallel computing and just-in-time compilation components of Jax to speed up the inference"
- [discussion] Meta-algorithm / importance scores from predictions: "VI-VS can be employed as a meta-algorithm to make the discoveries of existing methods more interpretable by constructing importance scores from their predictions."
- [readme] Raw counts requirement and scaling limit: [evidence span withheld — anchor status: not_found]
