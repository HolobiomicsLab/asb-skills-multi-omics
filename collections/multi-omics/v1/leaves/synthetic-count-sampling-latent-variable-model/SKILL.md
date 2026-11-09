---
name: synthetic-count-sampling-latent-variable-model
description: Use when performing conditional independence testing (CRT) of a gene
  feature g against a response Y in single-cell count data, where synthetic null replicates
  of x_g must be generated while conditioning on nuisance factors S such as batch
  and sequencing depth.
when_to_use_negative:
- The AnnData contains normalized expression rather than raw counts — VIVS explicitly
  requires raw counts
- The dataset has many thousands of genes without prior filtering — VIVS may not scale;
  use select_genes (e.g., n_top_genes=2000) first
- Data are chromatin accessibility (ATAC-seq) counts — the paper notes a generative
  model accounting for ATAC-seq sparsity may be required for valid p-values
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: VIVS
  role: Open-source Python package implementing the scVI-style generative model used
    to sample synthetic feature counts for the CRT
  repo: github.com/YosefLab/VIVS
- name: Jax
  role: Backend providing parallel computing and just-in-time compilation to speed
    up generative model inference and sampling
- name: scVI
  role: Reference VAE-based generative model whose decoder p_theta(x_g | z, s) produces
    normalized gene expression for synthetic sampling
- name: scanpy
  role: Provides the AnnData container holding raw counts, the response Y in obsm,
    and nuisance factors S
- name: PyTorch
  role: Alternative backend mentioned as a slower option compared to the Jax implementation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:ad67e6702548233b30a0040612083183ef101913557cea17c8496403c8e3061c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jax
  - PyTorch
  - Python
  - VI-VS
  - VIVS
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Synthetic count sampling from a latent-variable generative model

## Summary

Generates K i.i.d. synthetic replicates of a feature's expression counts by sampling a fixed posterior latent z-bar per cell with a scVI-style VAE and drawing from the decoder p_theta(x_g | z-bar, s), as required by the VI-VS conditional randomization test. The model is factorized as p(x, z | s) = p(z) * prod_g p(x_g | z, s) so the response y never enters the generative model.

## When to use

Use when performing conditional independence testing (CRT) of a gene feature g against a response Y in single-cell count data, where synthetic null replicates of x_g must be generated while conditioning on nuisance factors S such as batch and sequencing depth.

## When NOT to use

- The AnnData contains normalized expression rather than raw counts — VIVS explicitly requires raw counts
- The dataset has many thousands of genes without prior filtering — VIVS may not scale; use select_genes (e.g., n_top_genes=2000) first
- Data are chromatin accessibility (ATAC-seq) counts — the paper notes a generative model accounting for ATAC-seq sparsity may be required for valid p-values

## Inputs

- scanpy AnnData object containing raw (unnormalized) gene expression counts
- Response variable(s) Y stored in the AnnData obsm attribute (array or dataframe), keyed via feature_obsm_key
- Nuisance factors S (e.g., batch, sequencing depth, cell cycle) used as conditioning covariates
- Number of synthetic replicates K per tested feature

## Outputs

- K i.i.d. synthetic count vectors X-tilde_g^(k) per tested feature g, generated as p_theta(x_g | z-bar, s)
- Expression matrices with only gene g replaced by synthetic values, used for CRT statistic computation
- CRT p-values p_g = (1 + sum_k 1{T(X-tilde^(k), Y, S) <= T(X, Y, S)}) / (K + 1)

## How to apply

First load raw counts into a scanpy AnnData object (never normalized data) with the response Y in the obsm attribute and nuisance factors S (batch, sequencing depth, cell cycle), then split cells into a development set and a held-out validation set. Fit the VAE-based generative model with factorization p(x, z | s) = p(z) * prod_g p(x_g | z, s) on the development split only, so the model never sees the response y. For the tested feature g, encode each validation cell once to draw a single fixed posterior sample z-bar ~ q(z | x^n), then draw K i.i.d. synthetic vectors from the decoder via p_theta(x_g | z = z-bar, s), replacing only gene g in the expression matrix. Downstream, compute importance-score statistics T(X-tilde^(k), Y, S) for each replicate on the validation set and form the CRT p-value p_g = (1 + sum_k 1{T(X-tilde^(k), Y, S) <= T(X, Y, S)}) / (K + 1). The implementation relies on Jax for parallel computing and just-in-time compilation; scVI is the reference generative model for single-cell RNA counts.

## Related tools

- **VIVS** (Open-source Python package implementing the scVI-style generative model used to sample synthetic feature counts for the CRT) — github.com/YosefLab/VIVS
- **Jax** (Backend providing parallel computing and just-in-time compilation to speed up generative model inference and sampling)
- **scVI** (Reference VAE-based generative model whose decoder p_theta(x_g | z, s) produces normalized gene expression for synthetic sampling)
- **scanpy** (Provides the AnnData container holding raw counts, the response Y in obsm, and nuisance factors S)
- **PyTorch** (Alternative backend mentioned as a slower option compared to the Jax implementation)

## Examples

```
from vivs import VIVS; model = VIVS(adata, feature_obsm_key="my_obsm_key", xy_linear=False, xy_model_kwargs={"n_hidden": 8}); model.train_all()
```

## Evaluation signals

- Null simulation calibration: when x_g is conditionally independent of y, the empirical distribution of p_g must stochastically dominate the uniform distribution (valid p-value per Proposition 1)
- The generative model and importance scores must be trained only on the development split; p-values and cell scores must be computed on the held-out validation split
- Exactly one fixed posterior sample z-bar per validation cell is drawn; the K synthetic replicates must be i.i.d. draws from p_theta(x_g | z-bar, s) with only gene g replaced
- Verify the AnnData passed to the model contains raw counts, not normalized data, and that Y is present in the obsm attribute
- CRT p-value formula must yield values on the (K+1)-point grid {1/(K+1), ..., 1} with the +1 correction to avoid zero p-values

## Limitations

- VI-VS may not scale to many thousands of genes; gene filtering (e.g., select_genes with n_top_genes=2000) is recommended beforehand
- Spurious discoveries can arise from feedback loops, cell communication, or unobserved molecular species; VI-VS discoveries have no guarantees to be functional in the general case
- For chromatin accessibility data, a generative model properly accounting for ATAC-seq sparsity may be required to keep p-values valid
- Only assays where X corresponds to gene expression counts are currently supported by the VIVS implementation

## Evidence

- [other] In the VI-VS pipeline, the generative model and the importance scores are both trained on the development split of the data, while p-values (and cell scores) are computed on the held-out validation split.: [evidence span withheld — anchor status: not_found]
- [readme] Import data as AnnData with raw counts; response Y in obsm: "make sure than the anndata contains raw counts and not normalized data.

The response(s) $Y$ of interest are expected to be stored in the `obsm` attribute of the anndata object"
- [methods] Jax backend for speed: "Our implementation first relies on parallel computing and just-in-time compilation components of Jax to speed up the inference"
- [abstract] VI-VS uses deep generative models for conditional dependencies with FDR control: "It uses deep generative models to identify conditionally dependent features, with false discovery rate control."
- [intro] Calibrated FDR estimation via the conditional randomization test: "We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate."
