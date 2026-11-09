---
name: umap-spatial-cluster-visualization
description: Use when an integrated multi-omics embedding (SSGATE out_dims=30 output
  on h5ad/anndata inputs with obsm['spatial']) must be clustered and its clusters
  visually validated as spatially aggregated domains with clear boundaries.
when_to_use_negative:
- The AnnData object lacks spatial coordinates in obsm['spatial'] — SSGATE will treat
  it as single-cell multi-omics data and spatial aggregation cannot be assessed
- The goal is quantitative benchmarking against ground-truth labels — SCS_MT has none,
  so only qualitative acceptance checks apply
- Only one omics modality is available — this visualization skill presumes the joint
  transcriptome-proteome embedding from dual-path training
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0121
tools:
- name: SSGATE
  role: Produces the joint transcriptome-proteome embedding (out_dims=30) that is
    clustered and visualized; requires anndata inputs with X and obsm['spatial']
  repo: https://github.com/Linliu-Bioinf/SSGATE
- name: SCANPY
  role: Runs Leiden clustering on the integrated embedding (resolution 0.1–1.0) and
    renders UMAP and spatial cluster plots
- name: ClusterProfiler
  role: GO enrichment annotation of high-confidence Wilcoxon DEGs per cluster (Fig.
    4B) to support cluster interpretation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:64a28cec52ec43ea2b537f9470c658faa1c61bd625346edebbcadc142854f654
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# umap-spatial-cluster-visualization

## Summary

Clusters the SSGATE joint transcriptome-proteome embedding with Leiden (SCANPY, resolution 0.1–1.0) and renders the result as UMAP plots plus spatial maps overlaid on tissue coordinates, so that cluster quality can be judged by spatial aggregation and boundary sharpness (e.g. mouse thymus SCS_MT cortex/medulla).

## When to use

Use when an integrated multi-omics embedding (SSGATE out_dims=30 output on h5ad/anndata inputs with obsm['spatial']) must be clustered and its clusters visually validated as spatially aggregated domains with clear boundaries.

## When NOT to use

- The AnnData object lacks spatial coordinates in obsm['spatial'] — SSGATE will treat it as single-cell multi-omics data and spatial aggregation cannot be assessed
- The goal is quantitative benchmarking against ground-truth labels — SCS_MT has none, so only qualitative acceptance checks apply
- Only one omics modality is available — this visualization skill presumes the joint transcriptome-proteome embedding from dual-path training

## Inputs

- SSGATE integrated embedding (30-dimensional, from dual-path graph attention auto-encoder) stored in an AnnData object
- Spatial coordinates in anndata.obsm['spatial'] from the SCS_MT spatial transcriptomics-proteomics dataset (h5ad format)
- Leiden resolution values in the 0.1–1.0 range

## Outputs

- Leiden cluster labels per cell/spot
- UMAP plots showing cluster separation
- Spatial cluster maps overlaid on tissue coordinates (cortex/medulla-style anatomical domains)
- Optionally per-cluster Wilcoxon DEG tables and ClusterProfiler4 GO enrichment annotations

## How to apply

Take the joint embedding produced by ssgate.train (hidden_dims 128 per path, out_dims=30, n_epochs=300, lr=0.001, sigma=0.1) and cluster it with the Leiden algorithm in SCANPY, sweeping resolution across 0.1–1.0 to find the granularity that matches tissue anatomy. Visualize the clusters as UMAP plots to check cluster separation, and as spatial maps overlaid on the coordinates stored in anndata.obsm['spatial'] — if those coordinates are absent the data is silently handled as single-cell and no spatial map is possible. On the SCS_MT mouse thymus dataset (no ground-truth labels), accept the result qualitatively when UMAP clusters are well separated and the spatial map shows strong aggregation with clear boundaries reproducing the outer cortex and inner medulla. Optionally support the annotation by computing per-cluster Wilcoxon DEGs and GO enrichment (ClusterProfiler4) on filtered DEGs.

## Related tools

- **SSGATE** (Produces the joint transcriptome-proteome embedding (out_dims=30) that is clustered and visualized; requires anndata inputs with X and obsm['spatial']) — https://github.com/Linliu-Bioinf/SSGATE
- **SCANPY** (Runs Leiden clustering on the integrated embedding (resolution 0.1–1.0) and renders UMAP and spatial cluster plots)
- **ClusterProfiler** (GO enrichment annotation of high-confidence Wilcoxon DEGs per cluster (Fig. 4B) to support cluster interpretation)

## Examples

```
import scanpy as sc; sc.tl.leiden(adata, resolution=0.5, key_added='leiden'); sc.tl.umap(adata); sc.pl.umap(adata, color='leiden'); sc.pl.embedding(adata, basis='spatial', color='leiden')
```

## Evaluation signals

- UMAP plot shows well-separated clusters (no extensive inter-cluster mixing)
- Spatial cluster map displays strong spatial aggregation with clear tissue boundaries
- Spatial domains are consistent with the outer cortex and inner medulla of the mouse thymus (qualitative check, no ground truth)
- Leiden resolution sweep across 0.1–1.0 yields a stable granularity matching anatomical divisions
- Spatial coordinates were actually read (obsm['spatial'] present); otherwise output silently degrades to single-cell mode

## Limitations

- No ground-truth labels for SCS_MT, so validation is a qualitative acceptance check only
- SSGATE hyperparameters are defaults 'based on experience' rather than systematically derived, which can affect the embedding being visualized
- If anndata.X does not hold the expression profiles or obsm['spatial'] is missing, the data cannot be read correctly or is processed as single-cell multi-omics

## Evidence

- [readme] Spatial coordinates handling: "The spatial coordinates from spatial multi-omics data are required to be stored in `anndata.obsm['spatial']`, otherwise it will be processed as single-cell multi-omics data"
- [readme] Joint embedding is the clustering input: "After multiple rounds of training, the joint feature representation of the two omics can be obtained."
- [abstract] Method quality claim: "SSGATE shows better performance and stronger robustness than competitive methods and facilitates downstream analysis."
- [abstract] Spatial graph basis: "It can construct the neighborhood graphs based on single-cell expression profiles or spatial coordinates"
