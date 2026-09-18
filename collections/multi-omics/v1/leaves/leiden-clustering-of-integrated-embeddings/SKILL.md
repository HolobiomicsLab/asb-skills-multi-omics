---
name: leiden-clustering-of-integrated-embeddings
description: Use when you have a learned joint multi-omics embedding (e.g. SSGATE
  out_dims=30 output for transcriptomics-proteomics data in AnnData/h5ad) and need
  discrete cell clusters that can be checked for spatial aggregation against tissue
  anatomy.
when_to_use_negative:
- When the input is raw expression data rather than an integrated embedding — SSGATE
  graph construction and training must precede clustering
- When a ground-truth labeled dataset exists for supervised cell-type assignment —
  this workflow is a qualitative, unsupervised check
- When the goal is benchmarking integration quality with quantitative metrics; the
  thymus SCS_MT case has no ground-truth labels and relies on qualitative spatial-agreement
  checks
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: SSGATE
  role: Produces the joint transcriptome-proteome embedding (out_dims=30) that is
    clustered with Leiden
  repo: https://github.com/Linliu-Bioinf/SSGATE
- name: SCANPY
  role: Runs Leiden clustering (resolution 0.1–1.0) and UMAP visualization of the
    integrated embedding
- name: ClusterProfiler
  role: GO enrichment analysis on high-confidence DEGs (p-value and log fold change
    filters) to annotate cluster functions (Fig. 4B)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:552f68db73d082f6203f2c6374bd6bad89d2d0bb9a792c0e560af914fe20e2ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Linliu-Bioinf/SSGATE
  tools:
  - ClusterProfiler
  - MOFA
  - SCANPY
  - SSGATE
  - Seurat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# leiden-clustering-of-integrated-embeddings

## Summary

Cluster the joint transcriptome-proteome embedding produced by SSGATE with the Leiden algorithm (SCANPY) and evaluate the clusters as UMAP plots and spatial maps, as demonstrated on the SCS_MT mouse thymus spatial multi-omics data (resolution scanned 0.1–1.0).

## When to use

Use when you have a learned joint multi-omics embedding (e.g. SSGATE out_dims=30 output for transcriptomics-proteomics data in AnnData/h5ad) and need discrete cell clusters that can be checked for spatial aggregation against tissue anatomy.

## When NOT to use

- When the input is raw expression data rather than an integrated embedding — SSGATE graph construction and training must precede clustering
- When a ground-truth labeled dataset exists for supervised cell-type assignment — this workflow is a qualitative, unsupervised check
- When the goal is benchmarking integration quality with quantitative metrics; the thymus SCS_MT case has no ground-truth labels and relies on qualitative spatial-agreement checks

## Inputs

- Joint multi-omics embedding from SSGATE (30-dimensional, stored in AnnData)
- SCS_MT mouse thymus spatial transcriptomics-proteomics dataset (h5ad, spatial coordinates in anndata.obsm['spatial'])

## Outputs

- Leiden cluster assignments per cell/spatial spot
- UMAP plots of the integrated embedding colored by cluster
- Spatial cluster maps overlaid on tissue coordinates
- Per-cluster Wilcoxon DEGs and ClusterProfiler4 GO enrichment annotations

## How to apply

Take the joint embedding returned by ssgate.train (out_dims=30) from a dual-path graph attention auto-encoder trained with hidden_dims=128 per path, n_epochs=300, lr=0.001, sigma=0.1. Run Leiden clustering via SCANPY, scanning resolution over a 0.1–1.0 range to select a granularity that resolves the tissue's anatomical compartments. Visualize the clusters as UMAP plots to check cluster separation, and as spatial maps overlaid on anndata.obsm['spatial'] coordinates to check spatial aggregation. On the SCS_MT mouse thymus data, correct application yields clusters with clear boundaries consistent with the outer cortex and inner medulla; then compute per-cluster Wilcoxon DEGs and GO enrichment with ClusterProfiler4 (p-value and log fold change filters) to annotate cluster functions.

## Related tools

- **SSGATE** (Produces the joint transcriptome-proteome embedding (out_dims=30) that is clustered with Leiden) — https://github.com/Linliu-Bioinf/SSGATE
- **SCANPY** (Runs Leiden clustering (resolution 0.1–1.0) and UMAP visualization of the integrated embedding)
- **ClusterProfiler** (GO enrichment analysis on high-confidence DEGs (p-value and log fold change filters) to annotate cluster functions (Fig. 4B))

## Examples

```
import scanpy as sc; sc.pp.neighbors(adata, use_rep='ssgate'); sc.tl.umap(adata); sc.tl.leiden(adata, resolution=0.5, key_added='ssgate_leiden')
```

## Evaluation signals

- UMAP of the integrated embedding shows well-separated clusters
- Spatial cluster map displays strong spatial aggregation with clear boundaries
- Cluster boundaries align with the outer cortex and inner medulla regions of the mouse thymus (qualitative check, SCS_MT has no ground-truth labels)
- Leiden resolution in the 0.1–1.0 range yields stable, anatomically consistent cluster counts
- Per-cluster Wilcoxon DEGs yield coherent GO enrichment terms supporting distinct cluster functions

## Limitations

- Acceptance is a qualitative check: SCS_MT has no ground-truth labels, so spatial agreement with cortex/medulla is judged visually
- Hyperparameters (resolution range, SSGATE training settings) are set 'based on experience' as defaults rather than systematically derived
- Spatial information is only used if coordinates are stored in anndata.obsm['spatial']; otherwise data are processed as single-cell multi-omics

## Evidence

- [full_text] Cluster the integrated embedding with the Leiden algorithm (SCANPY, resolution 0.1–1.0 range) and visualize the clusters as UMAP plots and as spatial maps: [evidence span withheld — anchor status: not_found]
- [results] SSGATE-derived cell clusters display strong spatial aggregation with clear boundaries, highly consistent with the broad anatomical division of the mouse thymus.: [evidence span withheld — anchor status: not_found]
- [full_text] Compute per-cluster differentially expressed genes with the Wilcoxon test and run GO enrichment analysis on high-confidence DEGs: [evidence span withheld — anchor status: not_found]
- [readme] The spatial coordinates from spatial multi-omics data are required to be stored in `anndata.obsm['spatial']`: "The spatial coordinates from spatial multi-omics data are required to be stored in `anndata.obsm['spatial']`, otherwise it will be processed as single-cell multi-omics data"
- [discussion] Based on experience, we provide a set of hyperparameters suitable for most datasets as default settings: "Based on experience, we provide a set of hyperparameters suitable for most datasets as default settings"
