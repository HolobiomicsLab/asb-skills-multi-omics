---
name: null-simulation-pvalue-uniformity-checking
description: Use when validating that a CRT-style conditional independence testing
  pipeline produces valid, calibrated p-values before applying it to real single-cell
  data.
when_to_use_negative:
- The goal is to detect conditionally dependent features in real data rather than
  to validate p-value calibration — use the full VI-VS importance pipeline instead
- The testing procedure is not a conditional randomization test, so p-value uniformity
  under a simulated null would not certify validity via Proposition 1
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: VIVS
  role: Python package implementing the VI-VS conditional randomization test whose
    p-values are validated by the null simulation
  repo: github.com/YosefLab/VIVS
- name: Jax
  role: Backend used by VIVS for parallel computing and just-in-time compilation during
    model inference
- name: scVI
  role: Generative model used to produce synthetic samples X-tilde_g^(k) for the CRT
- name: scanpy
  role: AnnData container holding raw counts, response Y in obsm, and nuisance factors
    S
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:e1817b1006d9f8f0032c6db6863656097b33c4a742b2a071a8e14b67c6da514b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jax
  - Python
  - VI-VS
  - VIVS
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Null-simulation p-value uniformity checking

## Summary

Validation procedure for conditional randomization test p-values (as implemented in VI-VS): simulate a null case where the tested feature x_g is conditionally independent of the response y, then verify that the empirical distribution of p-values stochastically dominates the uniform distribution, confirming calibrated/valid p-values per Proposition 1.

## When to use

Use when validating that a CRT-style conditional independence testing pipeline produces valid, calibrated p-values before applying it to real single-cell data.

## When NOT to use

- The goal is to detect conditionally dependent features in real data rather than to validate p-value calibration — use the full VI-VS importance pipeline instead
- The testing procedure is not a conditional randomization test, so p-value uniformity under a simulated null would not certify validity via Proposition 1

## Inputs

- Simulated single-cell count data where x_g is conditionally independent of y, with nuisance factors S (batch, sequencing depth, cell cycle)
- Fitted VI-VS model (scVI-style generative model + importance score p_phi, e.g., linear regression or MLP with n_hidden=8)
- Number of synthetic replicates K used for the CRT

## Outputs

- Empirical distribution of null CRT p-values p_g
- Stochastic-dominance assessment of the null p-value distribution against Uniform(0,1)
- Pass/fail validation result recorded in a test report

## How to apply

Set up a null simulation in which the tested gene expression x_g is conditionally independent of the response y while preserving the nuisance structure S (e.g., batch, sequencing depth, cell cycle). Run the full VI-VS testing loop: fit the scVI-style generative model p(x, z | s) = p(z) * prod_g p(x_g | z, s) on the development split, draw one fixed posterior sample z-bar per validation cell, generate K synthetic replicates X-tilde_g^(k) from the decoder, and compute CRT p-values p_g = (1 + sum_k 1{T(X-tilde^(k), Y, S) <= T(X, Y, S)}) / (K + 1) using the importance score T(X, Y, S) = (1/N) sum_n -log p_phi(y^n | x^n, s^n). Then inspect the empirical distribution of the simulated null p-values: a valid test requires that the p-values stochastically dominate the uniform distribution (i.e., no enrichment of small p-values under the null), consistent with Proposition 1. Record pass/fail in a test report.

## Related tools

- **VIVS** (Python package implementing the VI-VS conditional randomization test whose p-values are validated by the null simulation) — github.com/YosefLab/VIVS
- **Jax** (Backend used by VIVS for parallel computing and just-in-time compilation during model inference)
- **scVI** (Generative model used to produce synthetic samples X-tilde_g^(k) for the CRT)
- **scanpy** (AnnData container holding raw counts, response Y in obsm, and nuisance factors S)

## Examples

```
from vivs import VIVS; model = VIVS(adata, feature_obsm_key='my_obsm_key', xy_linear=False, xy_model_kwargs={'n_hidden': 8}); model.train_all()
```

## Evaluation signals

- Under the simulated null (x_g independent of y given S), the empirical CDF of p_g lies at or below the diagonal — p-values stochastically dominate Uniform(0,1)
- No significant enrichment of small p-values (e.g., no excess below 0.05) in the null simulation histogram
- Each p_g is computed with the correct formula: (1 + sum_k 1{T(X-tilde^(k), Y, S) <= T(X, Y, S)}) / (K + 1), so values fall in [1/(K+1), 1]
- Generative model and importance score trained only on the development split; p-values computed only on the held-out validation split
- Pass/fail result explicitly recorded in the test report

## Limitations

- Validity holds under the null only if the generative model adequately matches the conditional distribution; for chromatin accessibility data, the article notes it 'may require the use of a generative model that properly accounts for ATAC-seq sparsity' to keep p-values valid
- Unobserved confounders, feedback loops, and unobserved molecular species can lead to spurious discoveries even with calibrated p-values

## Evidence

- [other] run a null simulation in which x_g is conditionally independent of y and verify that the empirical distribution of p_g stochastically dominates the uniform distribution (valid p-value per Proposition 1): [evidence span withheld — anchor status: not_found]
- [intro] We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate.: "We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate."
- [discussion] chromatin accessibility data may require the use of a generative model that property account for ATAC-seq sparsity: "chromatin accessibility data may require the use of a generative model that property account for ATAC-seq sparsity"
- [readme] In particular, make sure than the anndata contains raw counts and not normalized data.: "In particular, make sure than the anndata contains raw counts and not normalized data."
