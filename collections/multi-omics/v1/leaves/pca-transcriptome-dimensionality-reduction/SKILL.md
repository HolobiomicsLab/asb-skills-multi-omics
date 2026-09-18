---
name: pca-transcriptome-dimensionality-reduction
description: 'Use when preparing single-cell or spatially resolved transcriptome data
  for SSGATE neighborhood-graph construction: the expression matrix in anndata.X is
  high-dimensional and a ''X_pca'' feature matrix must exist before calling Cal_Nbrs_Net.'
when_to_use_negative:
- 'Computing neighbors from spatial coordinates: the feat=''spatial'' path uses anndata.obsm[''spatial'']
  directly and does not require PCA on the transcriptome'
- 'Proteomics data: the proteomic features are used directly as the feature matrix
  for neighbor calculation, not the transcriptome PCA representation'
- 'Integrating omics beyond transcriptome/proteome: the paper notes reasonable normalization
  and dimensionality reduction methods for other single-omics data are not yet available'
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCANPY
  role: Runs the PCA that reduces the transcriptomic profile to 200 dimensions before
    neighbor-set computation
- name: anndata
  role: Container format holding the expression profiles (anndata.X) and spatial coordinates
    (anndata.obsm['spatial']) that PCA is applied to
  repo: https://anndata.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:3bd8c4893bee7b3196aac6c21dd54806bfb8a7729578bfbf65a35a313cd6498c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MOFA
  - SCANPY
  - anndata
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# pca-transcriptome-dimensionality-reduction

## Summary

Reduces the high-dimensional transcriptome profile to 200 principal-component dimensions (stored as 'X_pca') using SCANPY, so that SSGATE's Cal_Nbrs_Net can compute Euclidean-distance-based neighbor sets for graph construction.

## When to use

Use when preparing single-cell or spatially resolved transcriptome data for SSGATE neighborhood-graph construction: the expression matrix in anndata.X is high-dimensional and a 'X_pca' feature matrix must exist before calling Cal_Nbrs_Net.

## When NOT to use

- Computing neighbors from spatial coordinates: the feat='spatial' path uses anndata.obsm['spatial'] directly and does not require PCA on the transcriptome
- Proteomics data: the proteomic features are used directly as the feature matrix for neighbor calculation, not the transcriptome PCA representation
- Integrating omics beyond transcriptome/proteome: the paper notes reasonable normalization and dimensionality reduction methods for other single-omics data are not yet available

## Inputs

- AnnData object with expression profiles in anndata.X (h5ad format)
- For spatial data: spatial coordinates in anndata.obsm['spatial'] (routed to the spatial graph path instead of PCA-based neighbors)

## Outputs

- AnnData with a 200-dimensional PCA-reduced transcriptomic profile stored as 'X_pca'
- Populated adata.uns['nbrs_net'] containing the masked distance-matrix neighborhood graph, saved as nbrs_net.h5ad

## How to apply

Preprocess the transcriptome with count-depth scaling and log1p transformation, select top highly variable genes, then run SCANPY PCA on the transcriptomic profile to reduce it to 200 dimensions, storing the result as 'X_pca' in the AnnData object. This reduced representation becomes the feat='X_pca' argument to Cal_Nbrs_Net, where Euclidean distances between cells in the 200-dimensional space generate the k=15 neighbor sets (or Radius model with rad_cutoff) used to assemble the transcriptome graph TG. The proteome and spatial-coordinate paths bypass this PCA step: proteomic features or anndata.obsm['spatial'] coordinates are used directly as the feature matrix. The resulting neighbor network is stored in adata.uns['nbrs_net'] and written to nbrs_net.h5ad.

## Related tools

- **SCANPY** (Runs the PCA that reduces the transcriptomic profile to 200 dimensions before neighbor-set computation)
- **anndata** (Container format holding the expression profiles (anndata.X) and spatial coordinates (anndata.obsm['spatial']) that PCA is applied to) — https://anndata.readthedocs.io/en/latest/

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('input.h5ad'); sc.pp.pca(adata, n_comps=200); adata = Cal_Nbrs_Net(adata, feat='X_pca', k_cutoff=15, model='KNN')
```

## Evaluation signals

- nbrs_net.h5ad exists and adata.uns['nbrs_net'] is populated for both modalities
- Each graph is an m-by-m masked distance matrix with each cell retaining 15 neighbors, matching the paper's reported default
- 'X_pca' exists in the AnnData obsm/layer slots with the expected 200-dimensional shape before Cal_Nbrs_Net is called
- Spatial data with coordinates in anndata.obsm['spatial'] is routed to the spatial graph path rather than the expression-PCA path

## Limitations

- Hyperparameters (e.g. the 200-dimension PCA setting) are set 'based on experience' as defaults rather than systematically derived
- Normalization and dimensionality reduction methods for integrating omics beyond transcriptome/proteome are not yet available
- If spatial coordinates are not stored in anndata.obsm['spatial'], the data cannot be read and will be processed as single-cell multi-omics data

## Evidence

- [methods] The neighborhood-set generation module first applies PCA to the high-dimensional transcriptome profile, reducing its dimensions to 200 before computing Euclidean-distance-based neighbor sets: [evidence span withheld — anchor status: not_found]
- [readme] The feature matrix used to calculate the neighbors. Can be either reduced coordinates ('X_pca') or spatial coordinates ('spatial').: "The feature matrix used to calculate the neighbors. Can be either reduced coordinates ('X_pca') or spatial coordinates ('spatial')."
- [readme] The expression profiles are required to be stored in anndata.X, otherwise the data cannot be read correctly.: "The expression profiles are required to be stored in anndata.X, otherwise the data cannot be read correctly."
- [methods] run PCA to reduce the transcriptomic profile to 200 dimensions using SCANPY: [evidence span withheld — anchor status: not_found]
