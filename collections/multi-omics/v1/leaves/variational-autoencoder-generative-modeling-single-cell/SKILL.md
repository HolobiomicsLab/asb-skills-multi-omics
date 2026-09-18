---
name: variational-autoencoder-generative-modeling-single-cell
description: Use when testing conditional independence between gene expression features
  and a response (protein, perturbation phenotype) in single-cell multiomics data
  while adjusting for nuisance factors like batch or sequencing depth.
when_to_use_negative:
- The AnnData X contains normalized rather than raw counts — VIVS explicitly requires
  raw counts
- The dataset has many thousands of genes without prior filtering with select_genes,
  since VIVS may not scale
- The data are chromatin accessibility (ATAC-seq) profiles, where a generative model
  properly accounting for sparsity has not yet been validated for VI-VS p-value validity
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: VIVS
  role: Python package implementing the VAE-based conditional independence testing
    pipeline; provides VIVS model class and select_genes filter
  repo: github.com/YosefLab/VIVS
- name: scVI
  role: VAE latent-variable generative model of gene counts whose decoder p_theta(x_g
    | z, s) supplies synthetic feature samples for the CRT
- name: scanpy
  role: Used to import data of interest as a scanpy AnnData object with raw counts
    and obsm responses
- name: Jax
  role: Backend providing parallel computing and just-in-time compilation to speed
    up inference
- name: PyTorch
  role: Alternative backend compared against during implementation (Jax offered a
    twofold speedup over it)
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_008
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:1da413b764956ff85c20fef1d4ffcc2ad9c960d448f929674749d6e629129a6c
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
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# variational-autoencoder-generative-modeling-single-cell

## Summary

Fit a scVI-style variational autoencoder (VAE) latent-variable generative model on raw single-cell counts to produce the factorization p(x, z | s) = p(z) * prod_g p(x_g | z, s), then use its encoder/decoder to sample synthetic features for conditional randomization tests (VI-VS) with calibrated FDR control.

## When to use

Use when testing conditional independence between gene expression features and a response (protein, perturbation phenotype) in single-cell multiomics data while adjusting for nuisance factors like batch or sequencing depth.

## When NOT to use

- The AnnData X contains normalized rather than raw counts — VIVS explicitly requires raw counts
- The dataset has many thousands of genes without prior filtering with select_genes, since VIVS may not scale
- The data are chromatin accessibility (ATAC-seq) profiles, where a generative model properly accounting for sparsity has not yet been validated for VI-VS p-value validity

## Inputs

- scanpy AnnData object with raw gene expression counts (X) and response variables Y stored in obsm (e.g., 224 surface proteins in CITE-seq, or perturbation annotations in Perturb-seq)
- Nuisance factor annotations S (batch assignments, sequencing depth, cell cycle)
- Optional: per-cell guide/perturbation annotations for ground-truth definition

## Outputs

- Per-feature conditional randomization test p-values
- Benjamini-Hochberg-thresholded feature selections at target FDR (e.g., alpha=0.1)
- Per-method FDR/TPR or per-protein association count tables (e.g., perturbational_screen_fdr_tpr.csv, cite_seq_pbmc_associations.csv)

## How to apply

Import the data as a scanpy AnnData object with raw counts (not normalized data), store the response Y in the obsm attribute, and filter genes with select_genes(adata, n_top_genes=2000) since VIVS may not scale to many thousands of genes. Fit the scVI-style VAE on the development split only, never exposing the response y to the generative model, conditioning on nuisance factors S (batch, sequencing depth, cell cycle) via xy_linear=False and xy_model_kwargs={'n_hidden': 8}, then call model.train_all(). For the tested feature g, draw one fixed posterior sample z-bar ~ q(z | x^n) per validation cell from the encoder, then generate K i.i.d. synthetic vectors X-tilde_g^(k) from the decoder via p_theta(x_g | z=z-bar, s), replacing only gene g. Compute the CRT p-value p_g = (1 + sum_k 1{T(X-tilde^(k), Y, S) <= T(X, Y, S)}) / (K + 1) using an importance model trained on development data and evaluated on the held-out validation split, then apply Benjamini-Hochberg at the target FDR (e.g., alpha=0.1).

## Related tools

- **VIVS** (Python package implementing the VAE-based conditional independence testing pipeline; provides VIVS model class and select_genes filter) — github.com/YosefLab/VIVS
- **scVI** (VAE latent-variable generative model of gene counts whose decoder p_theta(x_g | z, s) supplies synthetic feature samples for the CRT)
- **scanpy** (Used to import data of interest as a scanpy AnnData object with raw counts and obsm responses)
- **Jax** (Backend providing parallel computing and just-in-time compilation to speed up inference)
- **PyTorch** (Alternative backend compared against during implementation (Jax offered a twofold speedup over it))

## Examples

```
from vivs import VIVS, select_genes; adata = select_genes(adata, n_top_genes=2000); model = VIVS(adata, feature_obsm_key="my_obsm_key", xy_linear=False, xy_model_kwargs={"n_hidden": 8}); model.train_all()
```

## Evaluation signals

- Null simulation: when x_g is conditionally independent of y, the empirical distribution of p_g stochastically dominates the uniform distribution (valid p-value per Proposition 1)
- Estimated FDR from the calibrated VI-VS procedure is at or below the nominal alpha=0.1 threshold against t-test ground truth
- VI-VS achieves lower FDR than OLS and marginal tests (70% vs 95% in the Perturb-seq benchmark) while retaining a similar number of true positives
- Isotype negative-control proteins (Rat-IgG) show few or no detected associations
- p-values computed on the held-out validation split while generative and importance models were fitted only on the development split

## Limitations

- VIVS may not scale to many thousands of genes; gene filtering with select_genes is recommended
- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries
- Applicability to chromatin accessibility data not yet demonstrated; ATAC-seq sparsity may require a different generative model to keep p-values valid
- Cross-environment reproducibility of discoveries is only a proposed mitigation, not yet validated

## Evidence

- [other] For the tested feature g, draw one fixed posterior sample: [evidence span withheld — anchor status: not_found]
- [other] Fit VAE on development split without response: [evidence span withheld — anchor status: not_found]
- [readme] VIVS model initialization parameters: "model = VIVS(
    adata,
    feature_obsm_key="my_obsm_key",
    xy_linear=False,
    xy_model_kwargs={"n_hidden": 8}
)
model.train_all()"
- [readme] Gene filtering requirement before VIVS: "VIVS may not scale to many thousands of genes.
In such a case, it is recommended to filter the genes before running VIVS"
- [readme] Raw counts requirement in AnnData: "In particular, make sure than the anndata contains raw counts and not normalized data."
- [methods] Jax backend for speed: "Our implementation first relies on parallel computing and just-in-time compilation components of Jax to speed up the inference"
- [intro] Calibrated FDR estimation: "We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate."
