---
name: spatial-coordinate-neighborhood-graph-construction
description: Use when an AnnData object (expression in anndata.X, spatial coordinates
  in anndata.obsm['spatial']) must be converted into a masked-distance neighbor network
  stored in adata.uns['nbrs_net'] prior to SSGATE training.
when_to_use_negative:
- Input is not an AnnData object or expression profiles are not stored in anndata.X
  — the data cannot be read correctly
- Spatial coordinates are not in anndata.obsm['spatial'] — the sample will be routed
  to the single-cell path instead of using spatial information
- You need to integrate omics types beyond transcriptome/proteome — the paper notes
  reasonable normalization and dimensionality-reduction methods for other single-omics
  data are not yet available
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: SSGATE
  role: Provides the Cal_Nbrs_Net function (feat, rad_cutoff, k_cutoff, model='Radius'/'KNN')
    that computes and stores the neighbor network in adata.uns['nbrs_net']
  repo: github.com/Lv-BioInfo/SSGATE
- name: SCANPY
  role: Performs the PCA reduction of the transcriptomic profile to 200 dimensions
    ('X_pca') used as the feature matrix for neighbor calculation
- name: anndata
  role: 'Defines the required input/output container: expression in anndata.X, spatial
    coordinates in anndata.obsm[''spatial''], graph in uns[''nbrs_net''], saved as
    nbrs_net.h5ad'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:c19c7e9e57b5664e162f80c2d7ecb5c354b7f4e0b092d1aa5e3efb70b312c8a8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MOFA
  - SCANPY
  - SSGATE
  - anndata
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# spatial-coordinate-neighborhood-graph-construction

## Summary

Constructs the weighted neighborhood graphs TG and PG that SSGATE's dual-path graph attention auto-encoder consumes, by computing Euclidean-distance neighbor sets from spatial coordinates (anndata.obsm['spatial']) or from PCA-reduced expression profiles (200 dims via SCANPY), using the Cal_Nbrs_Net interface.

## When to use

Use when an AnnData object (expression in anndata.X, spatial coordinates in anndata.obsm['spatial']) must be converted into a masked-distance neighbor network stored in adata.uns['nbrs_net'] prior to SSGATE training.

## When NOT to use

- Input is not an AnnData object or expression profiles are not stored in anndata.X — the data cannot be read correctly
- Spatial coordinates are not in anndata.obsm['spatial'] — the sample will be routed to the single-cell path instead of using spatial information
- You need to integrate omics types beyond transcriptome/proteome — the paper notes reasonable normalization and dimensionality-reduction methods for other single-omics data are not yet available

## Inputs

- AnnData object per omics with expression profiles in anndata.X
- Spatial coordinates in anndata.obsm['spatial'] (for spatially resolved data)
- PCA-reduced feature matrix 'X_pca' (200 dimensions) in adata.obsm for single-cell data

## Outputs

- adata.uns['nbrs_net']: m-by-m masked Euclidean distance matrix (weighted graph TG/PG)
- nbrs_net.h5ad file carrying the constructed neighborhood graphs

## How to apply

Load each single-omics AnnData (transcriptome or proteome) and verify expression profiles sit in anndata.X and, for spatially resolved data, spatial coordinates in anndata.obsm['spatial'] — otherwise the sample is silently processed as single-cell data. Preprocess the transcriptome with count-depth scaling and log1p transformation, select highly variable genes, and run PCA via SCANPY down to 200 dimensions, passing feat='X_pca' (or feat='spatial' for coordinates). Call Cal_Nbrs_Net choosing model='Radius' (requires rad_cutoff) or model='KNN' (requires k_cutoff; the paper's default is 15 neighbors per cell), computing Euclidean distances between all cell pairs. Assemble the graphs as element-wise products of pairwise distance matrices with binary neighbor-indicator matrices, store the result in adata.uns['nbrs_net'], and save to nbrs_net.h5ad before the prune_net step and ssgate.train.

## Related tools

- **SSGATE** (Provides the Cal_Nbrs_Net function (feat, rad_cutoff, k_cutoff, model='Radius'/'KNN') that computes and stores the neighbor network in adata.uns['nbrs_net']) — github.com/Lv-BioInfo/SSGATE
- **SCANPY** (Performs the PCA reduction of the transcriptomic profile to 200 dimensions ('X_pca') used as the feature matrix for neighbor calculation)
- **anndata** (Defines the required input/output container: expression in anndata.X, spatial coordinates in anndata.obsm['spatial'], graph in uns['nbrs_net'], saved as nbrs_net.h5ad)

## Examples

```
adata = Cal_Nbrs_Net(adata, feat='X_pca', rad_cutoff=None, k_cutoff=None, model='Radius')
```

## Evaluation signals

- nbrs_net.h5ad exists after the step completes
- adata.uns['nbrs_net'] is populated for both transcriptome and proteome modalities
- Each graph is an m-by-m masked Euclidean distance matrix (distances times binary neighbor indicators)
- With model='KNN' and k_cutoff=15, every cell retains exactly 15 neighbors, matching the paper's default
- Using feat='spatial' on spatially resolved data yields a graph consistent with physical proximity, not expression similarity

## Limitations

- Hyperparameters are set 'based on experience' as defaults rather than systematically derived
- If spatial coordinates are missing from anndata.obsm['spatial'], the data is silently processed as single-cell multi-omics data
- rad_cutoff is mandatory for the Radius model and k_cutoff for the KNN model — passing None triggers failure
- Optimization of workflow, efficiency, and applicability to additional omics types is deferred to future work

## Evidence

- [readme] Before running SSGATE for multi-omics joint feature analysis, you must use the `Cal_Nbrs_Net` function to construct a neighbor network for a single omics data (transcriptome or proteome).: "Before running SSGATE for multi-omics joint feature analysis, you must use the `Cal_Nbrs_Net` function to construct a neighbor network"
- [readme] The spatial coordinates from spatial multi-omics data are required to be stored in `anndata.obsm['spatial']`, otherwise it will be processed as single-cell multi-omics data due to the spatial information cannot be read.: "The spatial coordinates from spatial multi-omics data are required to be stored in `anndata.obsm['spatial']`, otherwise it will be processed"
- [abstract] It can construct the neighborhood graphs based on single-cell expression profiles or spatial coordinates, enabling it to process single-cell data and utilize spatial information from spatially resolved data.: "It can construct the neighborhood graphs based on single-cell expression profiles or spatial coordinates"
- [readme] The parameters of Cal_Nbrs_Net are: adata, feat='X_pca', rad_cutoff=None, k_cutoff=None, model='Radius', verbose=True: "`model`: Specifies the type of model to use. Possible values are 'Radius' and 'KNN'."
