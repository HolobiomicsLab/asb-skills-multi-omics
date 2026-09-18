---
name: knn-neighborhood-graph-construction
description: Use when preparing single-cell or spatially resolved multi-omics AnnData
  objects for SSGATE training and you must build cell-cell neighborhood graphs from
  expression profiles ('X_pca') or spatial coordinates ('spatial').
when_to_use_negative:
- The data is not in AnnData format or expression profiles are not in anndata.X —
  SSGATE cannot read it correctly
- Integrating omics types beyond transcriptome/proteome — the paper notes reasonable
  normalization and dimensionality-reduction methods for other single-omics data are
  not yet available
- The input already contains a populated pruned neighborhood graph in adata.uns['nbrs_net']
  — reconstruction would duplicate work
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3316
tools:
- name: SSGATE
  role: Provides the Cal_Nbrs_Net and prune_net functions that compute Euclidean-distance
    neighbor sets and assemble the weighted neighborhood graphs (TG and PG) stored
    in adata.uns['nbrs_net'].
  repo: https://github.com/Linliu-Bioinf/SSGATE
- name: SCANPY
  role: Performs count-depth scaling with log1p, highly variable gene selection, and
    PCA reduction of the transcriptome to 200 dimensions before neighbor computation.
- name: Leiden
  role: Used in prune_net to cluster cells so that pruned edges connect only cells
    within the same Leiden cluster.
merged_aliases:
- spatial-neighborhood-graph-construction
merged_alias_records:
- alias: spatial-neighborhood-graph-construction
  slug: spatial-neighborhood-graph-construction
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_003
  - task_007
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:219947f78759c181ebdb217dcfdc856184b287285347047bedd5e1f745a04c37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Linliu-Bioinf/SSGATE
  tools:
  - ClusterProfiler
  - Leiden
  - MOFA
  - SCANPY
  - SSGATE
  - Seurat
  - anndata
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# knn-neighborhood-graph-construction

## Summary

Constructs per-modality neighborhood graphs (KNN or Radius) for SSGATE multi-omics integration by computing Euclidean-distance neighbor sets on PCA-reduced transcriptomic features, proteomic features, or spatial coordinates, storing the graph in adata.uns['nbrs_net'].

## When to use

Use when preparing single-cell or spatially resolved multi-omics AnnData objects for SSGATE training and you must build cell-cell neighborhood graphs from expression profiles ('X_pca') or spatial coordinates ('spatial').

## When NOT to use

- The data is not in AnnData format or expression profiles are not in anndata.X — SSGATE cannot read it correctly
- Integrating omics types beyond transcriptome/proteome — the paper notes reasonable normalization and dimensionality-reduction methods for other single-omics data are not yet available
- The input already contains a populated pruned neighborhood graph in adata.uns['nbrs_net'] — reconstruction would duplicate work

## Inputs

- AnnData object per omics modality (.h5ad) with expression matrix in anndata.X
- Spatial coordinates stored in anndata.obsm['spatial'] (for spatial multi-omics data)
- PCA-reduced transcriptomic feature matrix ('X_pca', 200 dimensions) or proteomic features after CLR transformation
- Parameters: feat ('X_pca' or 'spatial'), model ('KNN' or 'Radius'), k_cutoff (e.g. 15) or rad_cutoff

## Outputs

- AnnData object with neighborhood graph stored in adata.uns['nbrs_net'] (m-by-m masked Euclidean-distance matrix)
- Pruned neighborhood network after prune_net (edges restricted within Leiden cell populations)
- Saved graph file (e.g. nbrs_net.h5ad) for downstream SSGATE training

## How to apply

Load each omics modality as a separate AnnData object with expression in anndata.X and spatial coordinates (if any) in anndata.obsm['spatial']. Preprocess transcriptomics with count-depth scaling plus log1p, select highly variable genes, and run PCA to 200 dimensions via SCANPY; apply centered log-ratio transformation to proteomics. Call SSGATE's Cal_Nbrs_Net with feat='X_pca' for single-cell data or feat='spatial' for spatial data, choosing model='KNN' with k_cutoff (the paper's sweep found 15 neighbors optimal in all metrics including ARI on BMNC and SLN111_D1) or model='Radius' with rad_cutoff. Optionally prune the resulting network with prune_net so edges connect only cells in the same Leiden cluster. The graph is stored as an m-by-m masked Euclidean-distance matrix in adata.uns['nbrs_net'], which downstream graph attention auto-encoder training consumes.

## Related tools

- **SSGATE** (Provides the Cal_Nbrs_Net and prune_net functions that compute Euclidean-distance neighbor sets and assemble the weighted neighborhood graphs (TG and PG) stored in adata.uns['nbrs_net'].) — https://github.com/Linliu-Bioinf/SSGATE
- **SCANPY** (Performs count-depth scaling with log1p, highly variable gene selection, and PCA reduction of the transcriptome to 200 dimensions before neighbor computation.)
- **Leiden** (Used in prune_net to cluster cells so that pruned edges connect only cells within the same Leiden cluster.)

## Examples

```
adata = Cal_Nbrs_Net(adata, feat='X_pca', rad_cutoff=None, k_cutoff=15, model='KNN'); adata = prune_net(adata)
```

## Evaluation signals

- adata.uns['nbrs_net'] is populated for both omics modalities after Cal_Nbrs_Net
- Each cell retains exactly k_cutoff neighbors (15 in the paper's optimal setting), i.e. the graph is an m-by-m masked distance matrix with 15 neighbors per cell
- The feat parameter routes spatial AnnData objects to the spatial graph path (obsm['spatial']) rather than being processed as single-cell data
- After prune_net, every remaining edge connects two cells assigned to the same Leiden cluster
- Hyperparameter sweep validation: 15 neighbors yields optimal Purity, Homogeneity, ARI, and NMI on both BMNC and SLN111_D1 (Table 2)

## Limitations

- Hyperparameters (including the number of neighbors) are set 'based on experience' as defaults rather than systematically derived, so the optimal k may vary per dataset
- The Radius model requires rad_cutoff to be provided; KNN requires k_cutoff — omitting the relevant parameter fails
- Spatial coordinates must be in anndata.obsm['spatial']; if missing, spatial data is silently processed as single-cell multi-omics data

## Evidence

- [readme] The feature matrix used to calculate the neighbors. Can be either reduced coordinates ('X_pca') or spatial coordinates ('spatial').: "The feature matrix used to calculate the neighbors. Can be either reduced coordinates ('X_pca') or spatial coordinates ('spatial')."
- [readme] k_cutoff: Defines the number of neighbors when using the KNN model. This value is required if the KNN model is used.: "Defines the number of neighbors when using the KNN model. This value is required if the KNN model is used."
- [readme] The updated AnnData object, containing the calculated neighboring network information, is stored in adata.uns['nbrs_net'].: "containing the calculated neighboring network information, is stored in adata.uns['nbrs_net']"
- [abstract] It can construct the neighborhood graphs based on single-cell expression profiles or spatial coordinates, enabling it to process single-cell data and utilize spatial information.: "It can construct the neighborhood graphs based on single-cell expression profiles or spatial coordinates"
- [readme] This function is used to prune the adjacency network calculated by the Cal_Nbrs_Net function, ensuring that each edge only connects cells within the same cell population.: "ensuring that each edge only connects cells within the same cell population"
