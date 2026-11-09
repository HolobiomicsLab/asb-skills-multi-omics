---
name: crt-pvalue-computation-with-generative-models
description: Use when raw single-cell counts in a scanpy AnnData need calibrated FDR-controlled
  tests of conditional dependence between genes and response variables (e.g., protein
  levels in obsm), especially for nonlinear relationships.
when_to_use_negative:
- The AnnData X holds normalized expression rather than raw counts — VIVS requires
  raw counts.
- The task is marginal association testing (e.g., gene co-expression correlation),
  where conditional dependency testing would change the estimand and may miss marginal-only
  effects.
- The feature matrix has many thousands of genes without pre-filtering — per the README,
  VIVS may not scale to many thousands of genes, so select_genes is required first.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: VIVS
  role: Python package implementing the VI-VS conditional randomization test with
    scVI generative model; provides select_genes, VIVS, get_hier_importance
  repo: https://github.com/YosefLab/VIVS
- name: scVI
  role: Generative latent variable model producing the normalized gene expression
    under which CRT features are resampled
- name: Jax
  role: Backend for parallel computing and just-in-time compilation that speeds up
    VI-VS inference
- name: scanpy
  role: Provides the AnnData container format expected for raw counts and obsm responses
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:49ee140dfaac6278a1719a7caa558b27e3e895d119c039a88541a8f00c275159
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VIVS
  tools:
  - Jax
  - Python
  - VI-VS
  - VIVS
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# crt-pvalue-computation-with-generative-models

## Summary

Computes calibrated conditional-independence p-values for genes or gene groups by resampling features under a deep generative model (scVI) within the VI-VS conditional randomization test framework, then correcting with Benjamini-Hochberg. This enables FDR-controlled variable selection on single-cell count data with nonlinear interactions.

## When to use

Use when raw single-cell counts in a scanpy AnnData need calibrated FDR-controlled tests of conditional dependence between genes and response variables (e.g., protein levels in obsm), especially for nonlinear relationships.

## When NOT to use

- The AnnData X holds normalized expression rather than raw counts — VIVS requires raw counts.
- The task is marginal association testing (e.g., gene co-expression correlation), where conditional dependency testing would change the estimand and may miss marginal-only effects.
- The feature matrix has many thousands of genes without pre-filtering — per the README, VIVS may not scale to many thousands of genes, so select_genes is required first.

## Inputs

- scanpy AnnData object with raw gene expression counts in X
- Response variables Y stored in the AnnData obsm attribute (array or dataframe)
- Number of top genes to retain (e.g., n_top_genes=2000)
- n_clusters_list of hierarchical resolutions (e.g., [200, 300])
- K Monte Carlo samples per feature/group for the CRT

## Outputs

- Per-feature and per-group CRT p-values corrected with Benjamini-Hochberg
- Hierarchical importance results (res) from model.get_hier_importance
- Per-resolution FDR/power table (e.g., fdr_power_multiresolution.csv) and detected gene groups/clusters

## How to apply

Import the data as a scanpy AnnData object with raw counts in X and responses in an obsm key, filter to the top 2000 genes with the vivs select_genes function, then initialize VIVS with a scVI generative model and a nonlinear predictive model (xy_linear=False, xy_model_kwargs={'n_hidden': 8}) and call model.train_all(). To obtain p-values, call model.get_hier_importance, which computes conditional randomization test (CRT) p-values with K Monte Carlo samples per feature or gene group and corrects them with Benjamini-Hochberg; pass n_clusters_list (e.g., [200, 300]) to also test hierarchical gene clusters built from the gene-by-gene empirical correlation matrix of scVI-normalized expression. Score each resolution by computing FDR and power, treating a selected group as a true positive if it contains at least one gene-level true positive. The rationale is that the deep generative model conditions on technical and biological nuisance factors (batch effects, sequencing depth, cell cycle), giving theoretically calibrated FDR control without assuming linear relationships.

## Related tools

- **VIVS** (Python package implementing the VI-VS conditional randomization test with scVI generative model; provides select_genes, VIVS, get_hier_importance) — https://github.com/YosefLab/VIVS
- **scVI** (Generative latent variable model producing the normalized gene expression under which CRT features are resampled)
- **Jax** (Backend for parallel computing and just-in-time compilation that speeds up VI-VS inference)
- **scanpy** (Provides the AnnData container format expected for raw counts and obsm responses)

## Examples

```
from vivs import select_genes, VIVS; adata = select_genes(adata, n_top_genes=2000); model = VIVS(adata, feature_obsm_key='my_obsm_key', xy_linear=False, xy_model_kwargs={'n_hidden': 8}); model.train_all(); res = model.get_hier_importance(n_clusters_list=[200, 300])
```

## Evaluation signals

- Benjamini-Hochberg-corrected CRT p-values yield calibrated FDR: empirical FDR on the semi-synthetic PBMC experiment should stay near the nominal level.
- Power increases at coarser resolutions (200-cluster, 300-cluster) compared to gene-level testing, with detections consistent across resolutions.
- A selected gene group counts as a true positive only if it contains at least one gene-level true positive; FDR/power table (fdr_power_multiresolution.csv) should reflect this scoring.
- Model training completes via model.train_all() on the select_genes-filtered AnnData (top 2000 genes) without normalization errors.
- Hierarchical clusters are derived from the gene-by-gene empirical correlation matrix of scVI-normalized expression.

## Limitations

- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries.
- VIVS may not scale to many thousands of genes, so genes must be pre-filtered (e.g., select_genes with n_top_genes=2000).
- Applicability to chromatin accessibility data is not yet demonstrated and may require a generative model accounting for ATAC-seq sparsity for p-values to remain valid.
- Cross-environment reproducibility of discoveries is only a prospective mitigation, not yet validated.

## Evidence

- [abstract] It uses deep generative models to identify conditionally dependent features, with false discovery rate control.: "It uses deep generative models to identify conditionally dependent features, with false discovery rate control."
- [readme] To use VIVS in your project, import the data of intest in a scanpy AnnData object.: "To use VIVS in your project, import the data of intest in a scanpy AnnData object."
- [methods] We implemented VI-VS in a fast and scalable way that is available as an open-source Python package.: "We implemented VI-VS in a fast and scalable way that is available as an open-source Python package."
- [methods] Our implementation first relies on parallel computing and just-in-time compilation components of Jax to speed up the inference: "Our implementation first relies on parallel computing and just-in-time compilation components of Jax to speed up the inference"
- [discussion] VI-VS effectively addresses this issue by conditioning on these nuisance factors.: "VI-VS effectively addresses this issue by conditioning on these nuisance factors."
