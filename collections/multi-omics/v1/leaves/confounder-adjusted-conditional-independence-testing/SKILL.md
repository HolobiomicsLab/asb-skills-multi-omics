---
name: confounder-adjusted-conditional-independence-testing
description: Use when single-cell count data (scanpy AnnData with raw counts, response
  Y in obsm) must be tested for feature–response dependencies while adjusting for
  technical/biological nuisance factors such as batch, sequencing depth, or cell cycle.
when_to_use_negative:
- The AnnData contains normalized expression rather than raw counts — VIVS explicitly
  requires raw counts.
- The dataset has many thousands of genes without prior filtering — VIVS 'may not
  scale to many thousands of genes', so apply select_genes first or choose another
  method.
- The goal is guaranteed functional/causal validation of discoveries — VI-VS discoveries
  'have no guarantees to be functional', and feedback loops, cell communication, or
  unobserved molecular species can lead to spurious discoveries.
- The data is chromatin accessibility (ATAC-seq) — applicability is not yet demonstrated
  and may require a generative model accounting for ATAC-seq sparsity.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: VI-VS
  role: The framework implementing the CRT-based conditional independence testing
    with FDR control; provided as the VIVS Python package
  repo: github.com/YosefLab/VIVS
- name: VIVS
  role: Open-source Python package used to fit the generative model, train importance
    scores, and compute hierarchical feature importance
  repo: https://github.com/YosefLab/VIVS
- name: Jax
  role: Backend providing parallel computing and just-in-time compilation to speed
    up inference (GPU installation recommended)
- name: scVI
  role: Style of deep generative model used as the decoder/encoder producing synthetic
    feature samples p_theta(x_g | z, s)
- name: scanpy
  role: Provides the AnnData object format used to import raw counts and response
    variables
- name: GENIE3
  role: Example pairwise-relationship algorithm whose discoveries can be calibrated
    by VI-VS as a meta-algorithm/wrapper
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:7669b29cfce0426c19ebdcea0ee9954a76d747568dc688cc13536772d964abb2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GENIE3
  - Jax
  - Python
  - VI-VS
  - VIVS
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# confounder-adjusted-conditional-independence-testing

## Summary

Runs the VI-VS conditional randomization test (CRT) to test whether a feature g is conditionally independent of a response Y given nuisance factors S, using a scVI-style deep generative model to synthesize null replicates and importance scores to produce calibrated p-values with FDR control.

## When to use

Use when single-cell count data (scanpy AnnData with raw counts, response Y in obsm) must be tested for feature–response dependencies while adjusting for technical/biological nuisance factors such as batch, sequencing depth, or cell cycle.

## When NOT to use

- The AnnData contains normalized expression rather than raw counts — VIVS explicitly requires raw counts.
- The dataset has many thousands of genes without prior filtering — VIVS 'may not scale to many thousands of genes', so apply select_genes first or choose another method.
- The goal is guaranteed functional/causal validation of discoveries — VI-VS discoveries 'have no guarantees to be functional', and feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries.
- The data is chromatin accessibility (ATAC-seq) — applicability is not yet demonstrated and may require a generative model accounting for ATAC-seq sparsity.

## Inputs

- scanpy AnnData object with raw (unnormalized) gene expression counts in X
- Response variable(s) Y stored in the AnnData obsm attribute (array or dataframe), keyed via feature_obsm_key
- Nuisance factors S (e.g., batch, sequencing depth, cell cycle) as covariates
- Number of top genes for pre-filtering (e.g., n_top_genes=2000)

## Outputs

- Calibrated per-feature CRT p-values p_g valid under the null (stochastically dominating Uniform(0,1))
- Hierarchical/multiresolution feature importance results (e.g., from get_hier_importance) with FDR-controlled discoveries
- Cell-specific scores for individual cells

## How to apply

Load the data as a scanpy AnnData object containing raw counts, the response Y in the obsm attribute, and nuisance factors S, then split cells into a development set for model fitting and a held-out validation set for score evaluation. Fit a VAE-based generative model with factorization p(x, z | s) = p(z) * prod_g p(x_g | z, s) on the development set only, so it never sees y; optionally filter genes first with select_genes(adata, n_top_genes=2000) since VIVS may not scale to many thousands of genes. For the tested feature g, draw one fixed posterior sample z-bar ~ q(z | x^n) per validation cell with the encoder, then draw K i.i.d. synthetic vectors from the decoder p_theta(x_g | z=z-bar, s), replacing only gene g in the expression matrix. Train the importance-score model p_phi (linear regression or an MLP with n_hidden=8, set via xy_linear=False and xy_model_kwargs) on development data and compute T(X, Y, S) = (1/N) sum_n -log p_phi(y^n | x^n, s^n) for observed and each synthetic replicate on validation data. Compute the CRT p-value p_g = (1 + sum_k 1{T(X-tilde^(k), Y, S) <= T(X, Y, S)}) / (K + 1), and combine feature-level p-values across resolutions via the multiresolution/hierarchical procedure with FDR control.

## Related tools

- **VI-VS** (The framework implementing the CRT-based conditional independence testing with FDR control; provided as the VIVS Python package) — github.com/YosefLab/VIVS
- **VIVS** (Open-source Python package used to fit the generative model, train importance scores, and compute hierarchical feature importance) — https://github.com/YosefLab/VIVS
- **Jax** (Backend providing parallel computing and just-in-time compilation to speed up inference (GPU installation recommended))
- **scVI** (Style of deep generative model used as the decoder/encoder producing synthetic feature samples p_theta(x_g | z, s))
- **scanpy** (Provides the AnnData object format used to import raw counts and response variables)
- **GENIE3** (Example pairwise-relationship algorithm whose discoveries can be calibrated by VI-VS as a meta-algorithm/wrapper)

## Examples

```
from vivs import VIVS; model = VIVS(adata, feature_obsm_key='my_obsm_key', xy_linear=False, xy_model_kwargs={'n_hidden': 8}); model.train_all(); res = model.get_hier_importance(n_clusters_list=[100, 200])
```

## Evaluation signals

- Null simulation: when x_g is conditionally independent of y, the empirical distribution of p_g stochastically dominates the uniform distribution (valid p-value per Proposition 1); record pass/fail in the test report.
- Confirm the AnnData X matrix holds raw counts, not normalized values, before fitting.
- Confirm the response Y is present in the obsm attribute under the key passed as feature_obsm_key.
- Check that the generative model and importance scores were trained only on the development split, while p-values and cell scores were computed on the held-out validation split.
- Verify the reported p-value uses the (1 + #{T_k <= T_obs}) / (K + 1) form and that FDR control is reported for the multiresolution testing output.

## Limitations

- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries.
- VIVS may not scale to many thousands of genes; gene filtering (e.g., n_top_genes=2000) is recommended.
- VIVS currently only supports assays where X corresponds to gene expression counts; chromatin accessibility may require a generative model that properly accounts for ATAC-seq sparsity to keep p-values valid.
- No software version information is provided for the generative model or VI-VS implementation; cross-environment reproducibility of discoveries remains prospective.

## Evidence

- [abstract] It uses deep generative models to identify conditionally dependent features, with false discovery rate control.: "It uses deep generative models to identify conditionally dependent features, with false discovery rate control."
- [discussion] VI-VS effectively addresses this issue by conditioning on these nuisance factors.: "VI-VS effectively addresses this issue by conditioning on these nuisance factors."
- [readme] In particular, make sure than the anndata contains raw counts and not normalized data.: "In particular, make sure than the anndata contains raw counts and not normalized data."
- [readme] VIVS may not scale to many thousands of genes.: "VIVS may not scale to many thousands of genes."
- [methods] Our implementation first relies on parallel computing and just-in-time compilation components of Jax to speed up the inference: "Our implementation first relies on parallel computing and just-in-time compilation components of Jax to speed up the inference"
