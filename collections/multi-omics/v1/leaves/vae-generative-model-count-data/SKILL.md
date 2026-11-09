---
name: vae-generative-model-count-data
description: Use when analyzing single-cell count data (scanpy AnnData with raw counts)
  where calibrated conditional-independence testing and FDR-controlled feature selection
  are needed, especially with correlated features or count-distributed responses.
when_to_use_negative:
- The AnnData contains normalized rather than raw counts — VIVS explicitly requires
  raw counts and may produce invalid p-values otherwise
- The feature matrix has many thousands of genes without pre-filtering — VIVS may
  not scale, so genes must first be reduced (e.g., select_genes with n_top_genes=2000)
- Data are chromatin accessibility assays — ATAC-seq sparsity may require a different
  generative model for p-values to remain valid
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: VIVS
  role: Open-source Python package implementing the VAE-based generative model, CRT
    testing, and variable selection with FDR control
  repo: https://github.com/YosefLab/VIVS
- name: Jax
  role: Backend providing parallel computing and just-in-time compilation to speed
    up generative model inference
- name: scVI
  role: Generative model for single-cell RNA-seq whose normalized gene expression
    supplies the feature distribution for conditional independence testing
- name: scanpy
  role: Provides the AnnData object format used to import raw count data and store
    responses in obsm
- name: PyTorch
  role: Alternative backend considered but rejected because it offered only a twofold
    speedup comparison (Jax chosen)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:606a1a1ef21c28392856115d509d15caada09f8a25c72594b40f2f13203e93cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VIVS
  tools:
  - Jax
  - PyTorch
  - Python
  - VI-VS
  - VIVS
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# vae-generative-model-count-data

## Summary

Train a variational-inference deep generative model (scVI-style, Jax backend) on raw single-cell gene expression counts to serve as the feature distribution model for VI-VS conditional randomization tests, enabling calibrated p-values and false discovery rate control for conditionally dependent features.

## When to use

Use when analyzing single-cell count data (scanpy AnnData with raw counts) where calibrated conditional-independence testing and FDR-controlled feature selection are needed, especially with correlated features or count-distributed responses.

## When NOT to use

- The AnnData contains normalized rather than raw counts — VIVS explicitly requires raw counts and may produce invalid p-values otherwise
- The feature matrix has many thousands of genes without pre-filtering — VIVS may not scale, so genes must first be reduced (e.g., select_genes with n_top_genes=2000)
- Data are chromatin accessibility assays — ATAC-seq sparsity may require a different generative model for p-values to remain valid

## Inputs

- scanpy AnnData object with raw gene expression counts in X (not normalized data)
- Response variables Y stored in the AnnData obsm attribute (array or dataframe)
- Optional designated feature-obsm key for conditional features

## Outputs

- Trained VAE/VI-VS generative model over the count matrix (Jax backend)
- Per-feature conditional randomization test p-values from Monte Carlo randomized copies
- Benjamini-Hochberg-calibrated feature discoveries / hierarchical importance results (get_hier_importance output)

## How to apply

Import the data as a scanpy AnnData object ensuring the X matrix contains raw counts (not normalized data), with response variables Y stored in the obsm attribute. If the feature matrix spans many thousands of genes, first filter with select_genes (e.g., n_top_genes=2000) since VIVS may not scale to very large feature sets. Initialize the VIVS model on the AnnData, choosing xy_linear=False and an MLP importance model (xy_model_kwargs={'n_hidden': 8}) when nonlinear relationships or count-distributed responses are expected — as shown when Poisson-generated responses broke OLS calibration but VI-VS with an MLP importance score controlled FDR. Train with model.train_all(), which relies on Jax's parallel computing and just-in-time compilation for scalable inference; a PyTorch backend was avoided for speed reasons. The fitted generative model (e.g., scVI-normalized gene expression) then supports Monte Carlo CRT p-values per feature computed from randomized copies drawn from a fixed posterior sample, followed by Benjamini-Hochberg FDR control at target levels.

## Related tools

- **VIVS** (Open-source Python package implementing the VAE-based generative model, CRT testing, and variable selection with FDR control) — https://github.com/YosefLab/VIVS
- **Jax** (Backend providing parallel computing and just-in-time compilation to speed up generative model inference)
- **scVI** (Generative model for single-cell RNA-seq whose normalized gene expression supplies the feature distribution for conditional independence testing)
- **scanpy** (Provides the AnnData object format used to import raw count data and store responses in obsm)
- **PyTorch** (Alternative backend considered but rejected because it offered only a twofold speedup comparison (Jax chosen))

## Examples

```
from vivs import VIVS, select_genes; adata = select_genes(adata, n_top_genes=2000); model = VIVS(adata, feature_obsm_key='my_obsm_key', xy_linear=False, xy_model_kwargs={'n_hidden': 8}); model.train_all()
```

## Evaluation signals

- Under the null, computed CRT p-values stochastically dominate the uniform distribution
- Realized FDR curve tracks the target FDR level after Benjamini-Hochberg application (calibration check, e.g., as in Additional file 1: Figure S1 comparisons against OLS)
- Model remains calibrated when responses are generated from count distributions (e.g., Poisson) rather than Gaussian noise
- p-values remain valid under an alternative generative model for single-cell RNA-seq data (as reported in Additional file 1: Table S1)
- AnnData input validation: X holds raw counts and Y responses are present in obsm before training

## Limitations

- VIVS may not scale to many thousands of genes; genes must be filtered beforehand (e.g., n_top_genes=2000)
- Only assays where X corresponds to gene expression counts are supported
- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries
- No software version information is provided for the generative model or VI-VS implementation
- Applicability to chromatin accessibility data is not yet demonstrated and may require a generative model accounting for ATAC-seq sparsity

## Evidence

- [readme] raw counts requirement: "In particular, make sure than the anndata contains raw counts and not normalized data."
- [methods] Jax backend: "Our implementation first relies on parallel computing and just-in-time compilation components of Jax to speed up the inference"
- [methods] scVI generative model: "the normalized gene expression of the generative model, e.g., scVI [32]."
- [intro] calibrated FDR: "We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate."
- [supplementary] FDR comparison finding: [evidence span withheld — anchor status: not_found]
