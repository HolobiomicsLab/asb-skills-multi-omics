---
name: neighborhood-graph-construction-with-neighbor-pruning
description: 'Use when preparing normalized transcriptome/proteome AnnData for SSGATE:
  per-omics KNN graphs (e.g. k_cutoff=15 on X_pca) or radius graphs must be built
  and Leiden-pruned before training.'
when_to_use_negative:
- The AnnData lacks both obsm['X_pca'] and obsm['spatial'] — Cal_Nbrs_Net only accepts
  these two feat matrices for neighbor calculation.
- The chosen integrator builds its own graph internally (e.g. Seurat v5, which constructs
  a weighted nearest neighbor graph for data representation), making a separate SSGATE
  graph redundant.
- Input expression is not stored in anndata.X — non-anndata formats cannot be read
  correctly by SSGATE.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0602
tools:
- name: SSGATE
  role: Supplies Cal_Nbrs_Net (per-omics KNN/radius neighbor network written to adata.uns['nbrs_net'])
    and prune_net (same-population edge pruning)
  repo: https://github.com/Linliu-Bioinf/SSGATE.git
- name: SCANPY
  role: Upstream preprocessing (count-depth scaling + log1p normalization, HVG selection)
    and computation of the obsm['X_pca'] embedding passed as feat to Cal_Nbrs_Net
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:dda787b6f43f583da37bd75965e449460a2b96a756b5e10d89a6c8032720c682
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Linliu-Bioinf/SSGATE
  tools:
  - SCANPY
  - SSGATE
  - Seurat v5
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# neighborhood-graph-construction-with-neighbor-pruning

## Summary

Builds modality-specific neighborhood graphs for SSGATE multi-omics integration: Cal_Nbrs_Net constructs a KNN or radius graph from each omics' expression embedding (X_pca) or spatial coordinates, then prune_net removes edges between different cell populations so the dual-path graph attention auto-encoder receives cluster-consistent per-omics graphs.

## When to use

Use when preparing normalized transcriptome/proteome AnnData for SSGATE: per-omics KNN graphs (e.g. k_cutoff=15 on X_pca) or radius graphs must be built and Leiden-pruned before training.

## When NOT to use

- The AnnData lacks both obsm['X_pca'] and obsm['spatial'] — Cal_Nbrs_Net only accepts these two feat matrices for neighbor calculation.
- The chosen integrator builds its own graph internally (e.g. Seurat v5, which constructs a weighted nearest neighbor graph for data representation), making a separate SSGATE graph redundant.
- Input expression is not stored in anndata.X — non-anndata formats cannot be read correctly by SSGATE.

## Inputs

- Per-omics AnnData object (transcriptome or proteome) with normalized expression in anndata.X
- PCA embedding of expression profiles in anndata.obsm['X_pca'] (used as feat for neighbor calculation)
- For spatially resolved data: spatial coordinates stored in anndata.obsm['spatial']

## Outputs

- AnnData per omics with neighbor network stored in adata.uns['nbrs_net']
- Post-prune_net AnnData whose adata.uns['nbrs_net'] contains the pruned adjacency (edges only within the same cell population), ready for ssgate.train

## How to apply

Call Cal_Nbrs_Net once per modality on the normalized AnnData (transcriptome and proteome separately), passing feat='X_pca' for expression-based graphs (or feat='spatial' for spatially resolved coordinates) and choosing model='KNN' with k_cutoff — the Table 3 sensitivity sweep fixes the number of neighbors at 15 for both modalities — or model='Radius' with rad_cutoff; the adjacency is written to adata.uns['nbrs_net']. Then run prune_net(adata), which retains only edges connecting cells within the same cell population (Leiden cluster labels) and overwrites adata.uns['nbrs_net'], so each graph reflects cluster-consistent neighborhoods rather than spurious cross-population links. Because Cal_Nbrs_Net builds a network for a single omics at a time, repeat the build-and-prune cycle for the second modality before ssgate.train; the dual-path graph attention auto-encoder consumes one pruned graph per omics path. Expression must already sit in anndata.X and, for spatial inputs, coordinates in anndata.obsm['spatial'], otherwise the object cannot be read correctly or is silently treated as single-cell data.

## Related tools

- **SSGATE** (Supplies Cal_Nbrs_Net (per-omics KNN/radius neighbor network written to adata.uns['nbrs_net']) and prune_net (same-population edge pruning)) — https://github.com/Linliu-Bioinf/SSGATE.git
- **SCANPY** (Upstream preprocessing (count-depth scaling + log1p normalization, HVG selection) and computation of the obsm['X_pca'] embedding passed as feat to Cal_Nbrs_Net)

## Examples

```
adata = Cal_Nbrs_Net(adata, feat='X_pca', k_cutoff=15, model='KNN'); adata = prune_net(adata)
```

## Evaluation signals

- adata.uns['nbrs_net'] exists in each omics object after Cal_Nbrs_Net and is replaced by the pruned network after prune_net.
- With model='KNN' and k_cutoff=15, each cell's neighborhood size matches 15; with model='Radius', all neighbors lie within rad_cutoff.
- After pruning, no edge connects cells assigned to different Leiden cell populations.
- Separate graphs are produced for transcriptome and proteome (one Cal_Nbrs_Net + prune_net cycle per modality), not a single merged graph.
- Downstream check: SSGATE trained on these pruned graphs (300 epochs, out_dims=30, lr=0.001, balance=0.1) achieves the best Purity, Homogeneity Score, ARI and NMI on both BMNC and SLN111_D1 across Leiden resolutions 0.1–1.0, matching Table 3.

## Limitations

- Graph hyperparameters are defaults chosen 'based on experience', not systematically derived; k_cutoff/rad_cutoff may need per-dataset tuning.
- Spatial coordinates must be stored in anndata.obsm['spatial'] — if absent, the data are processed as single-cell multi-omics and the spatial information cannot be read.
- Cal_Nbrs_Net handles one omics at a time and expression must be in anndata.X; the paper notes that normalization and dimensionality-reduction methods for omics beyond transcriptome/proteome are not yet available.

## Evidence

- [readme] Per-omics neighbor network via Cal_Nbrs_Net before SSGATE analysis: "Before running SSGATE for multi-omics joint feature analysis, you must use the `Cal_Nbrs_Net` function to construct a neighbor network for a single omics data (transcriptome or proteome)."
- [readme] prune_net restricts edges to same cell population: "This function is used to prune the adjacency network calculated by the `Cal_Nbrs_Net` function, ensuring that each edge only connects cells within the same cell population."
- [readme] Spatial coordinates must be in anndata.obsm['spatial']: "The spatial coordinates from spatial multi-omics data are required to be stored in `anndata.obsm['spatial']`"
- [readme] Expression must be stored in anndata.X: "The expression profiles are required to be stored in anndata.X, otherwise the data cannot be read correctly"
- [abstract] Neighborhood graphs built from expression profiles or spatial coordinates: "It can construct the neighborhood graphs based on single-cell expression profiles or spatial coordinates"
- [discussion] Hyperparameters are experience-based defaults: "Based on experience, we provide a set of hyperparameters suitable for most datasets as default settings"
