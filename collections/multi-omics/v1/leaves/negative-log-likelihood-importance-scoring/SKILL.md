---
name: negative-log-likelihood-importance-scoring
description: 'Use when computing CRT test statistics for conditional independence
  testing in VI-VS: after generating synthetic feature replicates, to score observed
  vs. synthetic data on a held-out validation split.'
when_to_use_negative:
- The AnnData contains normalized expression rather than raw counts — VIVS requires
  raw counts
- The goal is marginal (not conditional) association testing, where simpler co-expression
  methods apply but lack FDR calibration
- The dataset has many thousands of genes without prior filtering (e.g., select_genes
  with n_top_genes=2000), as VIVS may not scale
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: VIVS
  role: Implements the importance scoring and CRT p-value computation; p_phi is configured
    via xy_model_kwargs (e.g., n_hidden=8) and xy_linear
  repo: github.com/YosefLab/VIVS
- name: Jax
  role: Backend providing parallel computing and just-in-time compilation to speed
    up inference
  repo: https://jax.readthedocs.io/en/latest/installation.html
- name: scVI
  role: Generative model whose decoder produces the synthetic replicates X-tilde_g^(k)
    that are scored by the importance model
- name: scanpy
  role: Provides the AnnData container for raw counts and the obsm-embedded response
    Y
- name: Python
  role: Runtime environment (>= 3.9) for installing and running the vivs package
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:33d850d3135243e4ade890a80521bbcaccdb91d893815cf6ad204580a466ecb8
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# negative-log-likelihood-importance-scoring

## Summary

Trains an importance-score model p_phi on development data and scores observed and synthetic replicates by the average negative log-likelihood T(X, Y, S) = (1/N) sum_n -log p_phi(y^n | x^n, s^n), which feeds the conditional randomization test p-value in the VI-VS pipeline.

## When to use

Use when computing CRT test statistics for conditional independence testing in VI-VS: after generating synthetic feature replicates, to score observed vs. synthetic data on a held-out validation split.

## When NOT to use

- The AnnData contains normalized expression rather than raw counts — VIVS requires raw counts
- The goal is marginal (not conditional) association testing, where simpler co-expression methods apply but lack FDR calibration
- The dataset has many thousands of genes without prior filtering (e.g., select_genes with n_top_genes=2000), as VIVS may not scale

## Inputs

- scanpy AnnData object with raw gene expression counts (X)
- Response variable Y stored in the AnnData obsm attribute (array or dataframe)
- Nuisance factors S (e.g., batch, sequencing depth, cell cycle) as covariates
- K synthetic replicates X-tilde_g^(k) drawn from p_theta(x_g | z=z-bar, s) with one fixed posterior sample z-bar per validation cell

## Outputs

- Test statistic T(X, Y, S) for observed data and each synthetic replicate on the validation split
- CRT p-value p_g = (1 + sum_k 1{T(X-tilde^(k), Y, S) <= T(X, Y, S)}) / (K + 1) per tested feature
- Cell-specific scores on the validation split

## How to apply

Fit the importance-score model p_phi (e.g., a linear regression or an MLP with n_hidden=8, controlled via xy_model_kwargs) on the development split, with the response Y taken from the AnnData obsm attribute and nuisance factors S (batch, sequencing depth, cell cycle) as covariates; the generative model is fit separately with a scVI-style factorization p(x, z | s) = p(z) * prod_g p(x_g | z, s). On the held-out validation split, compute T(X, Y, S) = (1/N) sum_n -log p_phi(y^n | x^n, s^n) for the observed data and for each of the K synthetic replicates X-tilde^(k) generated from the decoder. Compute the CRT p-value p_g = (1 + sum_k 1{T(X-tilde^(k), Y, S) <= T(X, Y, S)}) / (K + 1); smaller statistics for observed data than synthetics indicate conditional dependence of feature g on the response. Set xy_linear=False for nonlinear response-feature relationships.

## Related tools

- **VIVS** (Implements the importance scoring and CRT p-value computation; p_phi is configured via xy_model_kwargs (e.g., n_hidden=8) and xy_linear) — github.com/YosefLab/VIVS
- **Jax** (Backend providing parallel computing and just-in-time compilation to speed up inference) — https://jax.readthedocs.io/en/latest/installation.html
- **scVI** (Generative model whose decoder produces the synthetic replicates X-tilde_g^(k) that are scored by the importance model)
- **scanpy** (Provides the AnnData container for raw counts and the obsm-embedded response Y)
- **Python** (Runtime environment (>= 3.9) for installing and running the vivs package)

## Examples

```
from vivs import VIVS; model = VIVS(adata, feature_obsm_key='my_obsm_key', xy_linear=False, xy_model_kwargs={'n_hidden': 8}); model.train_all()
```

## Evaluation signals

- Under the null (x_g conditionally independent of y), the empirical distribution of p_g stochastically dominates the uniform distribution (valid p-value per Proposition 1)
- Observed-data statistic T(X, Y, S) is smaller than most synthetic replicate statistics when feature g is conditionally dependent on y
- The response Y is correctly read from the obsm attribute and the AnnData holds raw (not normalized) counts
- Training of the generative model and importance scores uses only the development split; p-values are computed exclusively on the held-out validation split
- n_hidden of the MLP importance model matches the configured value (e.g., 8) and xy_linear is set as intended

## Limitations

- Feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries
- In the general case, VI-VS discoveries have no guarantees to be functional
- VIVS may not scale to many thousands of genes; prior gene filtering (e.g., select_genes with n_top_genes=2000) is recommended
- No software version information is provided for the generative model or VI-VS implementation

## Evidence

- [methods] T-statistic definition: [evidence span withheld — anchor status: not_found]
- [methods] CRT p-value formula: [evidence span withheld — anchor status: not_found]
- [methods] Train/test split discipline: [evidence span withheld — anchor status: not_found]
- [readme] Model configuration: "model = VIVS(
    adata,
    feature_obsm_key="my_obsm_key",
    xy_linear=False,
    xy_model_kwargs={"n_hidden": 8}
)"
- [readme] Data format requirement: "In particular, make sure than the anndata contains raw counts and not normalized data."
