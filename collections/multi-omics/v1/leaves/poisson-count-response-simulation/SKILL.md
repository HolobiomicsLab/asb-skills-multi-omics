---
name: poisson-count-response-simulation
description: Use when validating FDR calibration of a conditional-independence/variable-selection
  method (e.g., VI-VS vs OLS) on semi-synthetic single-cell count data whose response
  is generated from a Poisson, not Gaussian, noise model.
when_to_use_negative:
- The response variable is modeled with Gaussian noise on a linear signal — use the
  paper's original Gaussian-noised semi-synthetic scenario instead of the Poisson
  variant.
- The goal is discovery on real data rather than calibration benchmarking — VI-VS
  discoveries carry no functional guarantees in the general case (feedback loops,
  cell communication, unobserved species can cause spurious findings).
- The count matrix has many thousands of genes — VIVS may not scale; filter first
  with select_genes (e.g., n_top_genes=2000).
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3170
tools:
- name: VI-VS
  role: 'Method under test: trains a VAE generative model (Jax backend) and computes
    CRT p-values with K randomized copies of each gene from a fixed posterior sample
    z-bar'
  repo: github.com/YosefLab/VIVS
- name: Jax
  role: Backend providing parallel computing and just-in-time compilation to speed
    up VI-VS inference
- name: scanpy
  role: Holds the simulated count matrix as a scanpy AnnData object with raw counts;
    response stored in obsm
- name: scVI
  role: Reference generative model for normalized gene expression used in the VI-VS
    framework
- name: Python
  role: Implementation language for the simulation, VIVS package, and OLS baseline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:b016cefdd0f8070ef37b99d6217702b41d0e781d58708131da0dd7f21674d756
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
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# poisson-count-response-simulation

## Summary

A semi-synthetic validation design from the VI-VS paper (Genome Biology, 2024) in which single-cell count features are simulated with correlated structure and the response y is drawn as Poisson counts conditionally dependent on a designated feature, replacing the Gaussian-noised linear response. It is used to test whether OLS-based conditional independence testing loses false discovery rate (FDR) calibration while VI-VS retains it.

## When to use

Use when validating FDR calibration of a conditional-independence/variable-selection method (e.g., VI-VS vs OLS) on semi-synthetic single-cell count data whose response is generated from a Poisson, not Gaussian, noise model.

## When NOT to use

- The response variable is modeled with Gaussian noise on a linear signal — use the paper's original Gaussian-noised semi-synthetic scenario instead of the Poisson variant.
- The goal is discovery on real data rather than calibration benchmarking — VI-VS discoveries carry no functional guarantees in the general case (feedback loops, cell communication, unobserved species can cause spurious findings).
- The count matrix has many thousands of genes — VIVS may not scale; filter first with select_genes (e.g., n_top_genes=2000).

## Inputs

- Semi-synthetic correlated gene expression count matrix (scanpy AnnData object with raw counts)
- Designated causal feature and Poisson conditional distribution for generating the response y
- Nuisance factor covariates for the OLS baseline regression

## Outputs

- CRT p-values per feature for VI-VS (MLP importance score) and OLS importance scores
- Realized-vs-target FDR comparison table for both methods at each Benjamini-Hochberg target FDR level
- FDR calibration plot mirroring Additional file 1: Figure S1

## How to apply

Simulate a semi-synthetic feature matrix of gene expression counts with correlated features, then draw the response y as Poisson counts conditionally dependent on a designated feature instead of Gaussian-noised linear signal. Fit the VI-VS pipeline by training the VAE latent-variable generative model on the count matrix with the Jax backend (VIVS package, xy_linear=False) and compute CRT p-values per feature using Monte Carlo with K randomized copies of each gene drawn from a fixed posterior sample z-bar. Fit the OLS baseline using prediction error from a linear regression of y on X and nuisance factors, with CRT p-values computed under the same K randomizations. Apply Benjamini-Hochberg to both p-value vectors across a grid of target FDR levels, record realized FDR per method, and plot realized versus target FDR mirroring Additional file 1: Figure S1. A method is judged calibrated when its null p-values stochastically dominate the uniform distribution and its realized FDR curve tracks the target level.

## Related tools

- **VI-VS** (Method under test: trains a VAE generative model (Jax backend) and computes CRT p-values with K randomized copies of each gene from a fixed posterior sample z-bar) — github.com/YosefLab/VIVS
- **Jax** (Backend providing parallel computing and just-in-time compilation to speed up VI-VS inference)
- **scanpy** (Holds the simulated count matrix as a scanpy AnnData object with raw counts; response stored in obsm)
- **scVI** (Reference generative model for normalized gene expression used in the VI-VS framework)
- **Python** (Implementation language for the simulation, VIVS package, and OLS baseline)

## Examples

```
from vivs import VIVS; model = VIVS(adata, feature_obsm_key="my_obsm_key", xy_linear=False); model.train_all()
```

## Evaluation signals

- Realized FDR curve of VI-VS tracks the diagonal (target FDR level) at every Benjamini-Hochberg target level, as in Additional file 1: Figure S1.
- OLS realized FDR exceeds the target level under Poisson-generated responses, demonstrating miscalibration relative to VI-VS.
- VI-VS null p-values stochastically dominate the uniform distribution.
- CRT p-values for both methods are computed with the same K randomized copies per gene so the comparison is fair.
- Simulated AnnData contains raw (unnormalized) counts, as required by VIVS.

## Limitations

- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries.
- VIVS may not scale to many thousands of genes; genes should be filtered (e.g., select_genes with n_top_genes=2000) beforehand.
- VIVS only supports assays where X corresponds to raw gene expression counts; normalized data are not supported.

## Evidence

- [supplementary] Poisson-response FDR result: [evidence span withheld — anchor status: not_found]
- [intro] CRT calibration claim: "We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate."
- [methods] Jax backend for inference: "Our implementation first relies on parallel computing and just-in-time compilation components of Jax to speed up the inference"
- [readme] Raw counts requirement: "In particular, make sure than the anndata contains raw counts and not normalized data."
- [readme] Gene filtering before VIVS: "VIVS may not scale to many thousands of genes."
