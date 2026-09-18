---
name: annndata-uns-attribute-management
description: Use when an AnnData-based pipeline computes derived per-dataset artifacts
  — e.g. SSGATE's neighborhood graphs from 'X_pca' or 'spatial' features — that must
  persist across pipeline steps such as Cal_Nbrs_Net, prune_net, and train.
when_to_use_negative:
- Data is not in AnnData format — SSGATE requires anndata inputs; expression profiles
  must be in anndata.X or the data cannot be read correctly
- Spatial coordinates are absent or not stored in anndata.obsm['spatial'] — the object
  will be processed as single-cell multi-omics data because spatial information cannot
  be read, so no spatially-aware uns graph is appropriate
- The derived artifact is not graph/network state consumed by later pipeline steps
  (e.g. plain per-cell metadata) — other AnnData slots (obs, obsm) are the intended
  locations
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: anndata
  role: Container format whose `uns` unstructured-annotation attribute holds the SSGATE
    neighbor network across Cal_Nbrs_Net, prune_net, and train steps
  repo: https://anndata.readthedocs.io/en/latest/
- name: SSGATE
  role: Provides the Cal_Nbrs_Net and prune_net functions that write nbrs_net into
    adata.uns and the train function that reads it
  repo: https://github.com/Linliu-Bioinf/SSGATE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:16bc38d605048e865f3cffbc5138b32c6a746034f7725ef874b3a7d56f4d4261
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# annndata-uns-attribute-management

## Summary

Managing derived analysis artifacts in the AnnData `uns` attribute, as practiced by SSGATE, which stores the computed neighborhood network (nbrs_net) in `adata.uns['nbrs_net']` after construction and pruning so downstream training stages can consume the graph.

## When to use

Use when an AnnData-based pipeline computes derived per-dataset artifacts — e.g. SSGATE's neighborhood graphs from 'X_pca' or 'spatial' features — that must persist across pipeline steps such as Cal_Nbrs_Net, prune_net, and train.

## When NOT to use

- Data is not in AnnData format — SSGATE requires anndata inputs; expression profiles must be in anndata.X or the data cannot be read correctly
- Spatial coordinates are absent or not stored in anndata.obsm['spatial'] — the object will be processed as single-cell multi-omics data because spatial information cannot be read, so no spatially-aware uns graph is appropriate
- The derived artifact is not graph/network state consumed by later pipeline steps (e.g. plain per-cell metadata) — other AnnData slots (obs, obsm) are the intended locations

## Inputs

- AnnData object with expression profiles in anndata.X
- Spatial coordinates stored in anndata.obsm['spatial'] (for spatial multi-omics data)
- Feature matrix choice: 'X_pca' (PCA-reduced transcriptome) or 'spatial' coordinates
- Cal_Nbrs_Net parameters: feat, rad_cutoff (Radius model), k_cutoff (KNN model), model='Radius'/'KNN', verbose

## Outputs

- Updated AnnData object with neighbor network stored in adata.uns['nbrs_net']
- Pruned adjacency network in uns after prune_net, edges restricted to cells within the same cell population

## How to apply

In SSGATE, the Cal_Nbrs_Net function computes a neighbor network for a single omics modality (transcriptome or proteome) from a feature matrix given via `feat` — either reduced coordinates ('X_pca') or spatial coordinates ('spatial') — using either the Radius model (with `rad_cutoff`) or the KNN model (with `k_cutoff`). The function returns the updated AnnData object with the calculated neighboring network stored in `adata.uns['nbrs_net']`. The subsequent prune_net function prunes the adjacency network so each edge only connects cells within the same cell population, and again writes the pruned network back into the `uns` attribute, keeping `uns['nbrs_net']` the single authoritative location for graph state. This in-place AnnData convention lets the training step (`ssgate.train(adata_st, adata_sp, ...)`) read the graphs without extra file plumbing; correct application means each Cal_Nbrs_Net/prune_net call ends with a populated, m-by-m masked-distance structure in uns for each modality.

## Related tools

- **anndata** (Container format whose `uns` unstructured-annotation attribute holds the SSGATE neighbor network across Cal_Nbrs_Net, prune_net, and train steps) — https://anndata.readthedocs.io/en/latest/
- **SSGATE** (Provides the Cal_Nbrs_Net and prune_net functions that write nbrs_net into adata.uns and the train function that reads it) — https://github.com/Linliu-Bioinf/SSGATE

## Examples

```
adata = Cal_Nbrs_Net(adata, feat='X_pca', rad_cutoff=None, k_cutoff=15, model='KNN'); adata = prune_net(adata)
```

## Evaluation signals

- After Cal_Nbrs_Net, adata.uns['nbrs_net'] is populated (not None/empty) for each omics modality
- The stored graph is an m-by-m masked distance matrix with each cell retaining its neighbor set (paper default k=15 neighbors per cell under KNN; rad_cutoff radius under Radius model)
- After prune_net, each edge only connects cells within the same cell population, and the pruned network remains in uns['nbrs_net']
- The train call ssgate.train(adata_st, adata_sp, ...) runs without key errors, confirming uns['nbrs_net'] was correctly attached to both AnnData objects
- For spatial data, obsm['spatial'] was present and used (feat='spatial'); otherwise the object would have been silently processed as single-cell data

## Limitations

- Hyperparameters such as rad_cutoff and k_cutoff are provided 'based on experience' as defaults rather than systematically derived
- If spatial coordinates are missing from obsm['spatial'], the data is processed as single-cell multi-omics because the spatial information cannot be read
- Normalization and dimensionality reduction methods for omics beyond transcriptome/proteome are not yet available, limiting what can be routed through this graph-construction/uns workflow

## Evidence

- [readme] Cal_Nbrs_Net returns updated AnnData with nbrs_net in uns: "The updated AnnData object, containing the calculated neighboring network information, is stored in adata.uns['nbrs_net']."
- [readme] prune_net stores pruned adjacency network in uns: "After pruning, the updated adjacency network will be stored in the uns attribute of the AnnData object."
- [readme] Expression profiles must be in anndata.X: "The expression profiles are required to be stored in anndata.X, otherwise the data cannot be read correctly."
- [readme] Spatial coordinates must be in obsm['spatial']: "The spatial coordinates from spatial multi-omics data are required to be stored in `anndata.obsm['spatial']`"
- [abstract] Abstract: neighborhood graphs from expression profiles or spatial coordinates: "It can construct the neighborhood graphs based on single-cell expression profiles or spatial coordinates"
