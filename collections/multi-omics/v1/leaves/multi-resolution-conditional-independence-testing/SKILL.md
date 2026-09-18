---
name: multi-resolution-conditional-independence-testing
description: Use when running VI-VS on single-cell count data (e.g. PBMC with a scanpy
  AnnData of raw counts) and you want to report discoveries at multiple feature granularities
  (e.g. 200-, 300-cluster and per-gene) with calibrated FDR and power per resolution.
when_to_use_negative:
- The AnnData X contains normalized expression instead of raw counts — VIVS only supports
  assays where X corresponds to gene expression counts
- 'The data are chromatin accessibility (ATAC-seq): applicability is not yet demonstrated
  and p-values may not remain valid unless the generative model accounts for ATAC-seq
  sparsity'
- The dataset has many thousands of genes without prior filtering — VIVS may not scale;
  genes should first be reduced with vivs.select_genes (e.g. n_top_genes=2000)
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: VIVS
  role: Python package providing the VIVS model, gene filtering via select_genes,
    hierarchical clustering of features, and get_hier_importance for multi-resolution
    CRT testing with BH correction
  repo: github.com/YosefLab/VIVS
- name: scanpy
  role: Imports the data of interest as a scanpy AnnData object with raw counts; response
    Y stored in obsm
- name: scVI
  role: Deep generative latent variable model whose normalized gene expression underlies
    VI-VS's generative component and the empirical correlation matrix used for hierarchical
    clustering
- name: Jax
  role: Backend used by the VIVS implementation for parallel computing and just-in-time
    compilation to speed up inference; must be installed with GPU support
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:b503aad496d686c07eaa0d38225fa70e83d05f1b4751308131db4c370adeb128
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# multi-resolution-conditional-independence-testing

## Summary

Applies VI-VS's multiresolution testing to compute calibrated conditional randomization test (CRT) p-values for individual genes and for hierarchical clusters of genes at several granularities, so that signals missed at the single-gene level are recovered by grouping conditionally dependent genes together while false discovery rate remains controlled.

## When to use

Use when running VI-VS on single-cell count data (e.g. PBMC with a scanpy AnnData of raw counts) and you want to report discoveries at multiple feature granularities (e.g. 200-, 300-cluster and per-gene) with calibrated FDR and power per resolution.

## When NOT to use

- The AnnData X contains normalized expression instead of raw counts — VIVS only supports assays where X corresponds to gene expression counts
- The data are chromatin accessibility (ATAC-seq): applicability is not yet demonstrated and p-values may not remain valid unless the generative model accounts for ATAC-seq sparsity
- The dataset has many thousands of genes without prior filtering — VIVS may not scale; genes should first be reduced with vivs.select_genes (e.g. n_top_genes=2000)

## Inputs

- scanpy AnnData object with raw gene expression counts (X must be raw counts, not normalized)
- Response variable(s) Y stored in the AnnData obsm attribute (array or dataframe), referenced via feature_obsm_key
- Number of clusters per resolution, e.g. n_clusters_list=[200, 300] plus per-gene resolution
- Number K of Monte Carlo samples per CRT feature/group

## Outputs

- Hierarchical importance results object from model.get_hier_importance mapping each resolution's features/groups to calibrated CRT p-values (BH-corrected)
- Per-resolution FDR and power table written to fdr_power_multiresolution.csv
- Record of detected gene groups per resolution (e.g. clusters detected only at coarser resolution), optionally visualized with model.plot_hier_importance

## How to apply

First train the VIVS model on the scanpy AnnData (generative model: scVI latent variable model; predictive model with xy_linear=False and xy_model_kwargs={'n_hidden': 8}) via model.train_all(). Then cluster genes with fast agglomerative hierarchical clustering on the gene-by-gene empirical correlation matrix computed from the scVI-normalized expression, and call model.get_hier_importance with n_clusters_list covering the desired coarser resolutions (e.g. [200, 300]) plus the individual gene-level resolution. For each feature or group, compute a CRT p-value using K Monte Carlo samples and correct all p-values with the Benjamini-Hochberg procedure. Score each resolution by computing FDR and power, treating a selected group as a true positive if it contains at least one gene-level true positive; coarser resolutions typically yield more discoveries while maintaining calibrated FDR, and detections should be largely consistent across resolutions (e.g. clusters 5 and 6 in the semi-synthetic PBMC experiment contained relevant genes detected only at coarser resolutions). Write the per-resolution FDR/power table to fdr_power_multiresolution.csv and record the detected gene groups alongside it.

## Related tools

- **VIVS** (Python package providing the VIVS model, gene filtering via select_genes, hierarchical clustering of features, and get_hier_importance for multi-resolution CRT testing with BH correction) — github.com/YosefLab/VIVS
- **scanpy** (Imports the data of interest as a scanpy AnnData object with raw counts; response Y stored in obsm)
- **scVI** (Deep generative latent variable model whose normalized gene expression underlies VI-VS's generative component and the empirical correlation matrix used for hierarchical clustering)
- **Jax** (Backend used by the VIVS implementation for parallel computing and just-in-time compilation to speed up inference; must be installed with GPU support)

## Examples

```
from vivs import VIVS; model = VIVS(adata, feature_obsm_key='my_obsm_key', xy_linear=False, xy_model_kwargs={'n_hidden': 8}); model.train_all(); res = model.get_hier_importance(n_clusters_list=[200, 300])
```

## Evaluation signals

- Benjamini-Hochberg-corrected CRT p-values yield estimated FDR at or below the nominal target at every resolution (calibrated FDR)
- Number of discoveries at coarser resolutions (e.g. 200- and 300-cluster) is greater than or equal to the per-gene resolution, without FDR inflation
- Detected groups are consistent across resolutions; any cluster detected only at coarser resolution should contain at least one gene-level true positive when scored (e.g. clusters 5 and 6 in the semi-synthetic PBMC experiment)
- Per-resolution FDR/power table exists as fdr_power_multiresolution.csv with one row per resolution and accompanying record of detected gene groups
- Empirical correlation matrix for clustering is computed from scVI-normalized expression, not raw counts

## Limitations

- In the general case, VI-VS discoveries have no guarantees to be functional: feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries
- VIVS may not scale to many thousands of genes; gene filtering with select_genes is recommended before running
- VIVS only supports assays where X corresponds to gene expression counts; chromatin accessibility data may require a generative model that properly accounts for ATAC-seq sparsity to keep p-values valid
- No software version information for the generative model or VI-VS implementation is provided in the article section

## Evidence

- [discussion] Multiresolution testing procedure identifying individual genes and feature groups with conditionally dependent features: "To address this risk, we proposed a multiresolution testing procedure."
- [methods] Multi-resolution importance call in VIVS: [evidence span withheld — anchor status: not_found]
- [methods] Gene filtering before running VIVS: [evidence span withheld — anchor status: not_found]
- [readme] Raw counts requirement in AnnData input: "In particular, make sure than the anndata contains raw counts and not normalized data."
- [readme] n_clusters_list meaning in get_hier_importance: "Here, `n_clusters_list` is a list of the number of clusters to consider for the hierarchical clustering of the features."
