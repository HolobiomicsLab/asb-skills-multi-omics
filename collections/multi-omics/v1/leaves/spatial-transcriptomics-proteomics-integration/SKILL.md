---
name: spatial-transcriptomics-proteomics-integration
description: Use when you have paired spatial transcriptomics and proteomics data
  in AnnData (h5ad) format with coordinates in obsm['spatial'] and need clusters that
  respect spatial tissue structure.
when_to_use_negative:
- Input is single-cell (non-spatial) multi-omics without spatial coordinates — SSGATE
  can handle it, but the spatial-information workflow (obsm['spatial'], spatial neighborhood
  graphs) does not apply
- Integration of omics types beyond transcriptome/proteome — the authors note normalization
  and dimensionality-reduction methods for other single-omics data are not yet available
- You need strictly derived hyperparameters — defaults were set 'based on experience'
  rather than systematically optimized
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SSGATE
  role: Dual-path graph attention auto-encoder that constructs neighborhood graphs
    from spatial coordinates and performs self-supervised multi-omics integration
  repo: https://github.com/Linliu-Bioinf/SSGATE
- name: SCANPY
  role: Leiden clustering of the integrated embedding and UMAP/spatial visualization
- name: ClusterProfiler
  role: GO enrichment analysis on high-confidence DEGs to annotate cluster functions
- name: MOFA
  role: Competitive/alternative unsupervised multi-omics integration method referenced
    for comparison
- name: Seurat
  role: Network-based competitive integration method (weighted nearest neighbor graph)
    referenced for comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:380776638a8625a83e849becc3f7692522a0435afc66cf5fe376485475b617d9
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spatial-transcriptomics-proteomics-integration

## Summary

Integrate paired spatial transcriptomics and proteomics data with SSGATE, a dual-path graph attention auto-encoder that builds neighborhood graphs from spatial coordinates and self-supervised learning to yield a joint embedding that resolves spatial domains (e.g., mouse thymus cortex/medulla).

## When to use

Use when you have paired spatial transcriptomics and proteomics data in AnnData (h5ad) format with coordinates in obsm['spatial'] and need clusters that respect spatial tissue structure.

## When NOT to use

- Input is single-cell (non-spatial) multi-omics without spatial coordinates — SSGATE can handle it, but the spatial-information workflow (obsm['spatial'], spatial neighborhood graphs) does not apply
- Integration of omics types beyond transcriptome/proteome — the authors note normalization and dimensionality-reduction methods for other single-omics data are not yet available
- You need strictly derived hyperparameters — defaults were set 'based on experience' rather than systematically optimized

## Inputs

- Paired spatial transcriptomics and proteomics data in AnnData (h5ad) format, expression in anndata.X
- Spatial coordinates stored in anndata.obsm['spatial']
- SCS_MT mouse thymus spatial transcriptomics-proteomics dataset (available via BGI cloud disk linked in the SSGATE README)

## Outputs

- 30-dimensional joint transcriptome-proteome cell embedding from ssgate.train
- Leiden cluster assignments across resolutions 0.1–1.0 with UMAP and spatial cluster maps
- Per-cluster Wilcoxon DEGs and GO enrichment annotations from ClusterProfiler4

## How to apply

Load the h5ad AnnData with expression in anndata.X and spatial coordinates in anndata.obsm['spatial']; preprocess the transcriptome with count-depth scaling plus log1p and top highly-variable-gene selection, and the proteome with centered log-ratio normalization. Construct a neighbor network per omics with Cal_Nbrs_Net (feat='spatial', model='KNN', k_cutoff=15), then prune it with prune_net so each edge connects cells within the same Leiden population. Train the dual-path graph attention auto-encoder via ssgate.train with hidden_dims=128 per path, out_dims=30, n_epochs=300, lr=0.001, sigma=0.1 to get a 30-dimensional joint transcriptome-proteome embedding. Cluster the embedding with Leiden (SCANPY, resolution swept 0.1–1.0) and visualize UMAP plus spatial maps; annotate clusters with Wilcoxon DEGs and ClusterProfiler4 GO enrichment.

## Related tools

- **SSGATE** (Dual-path graph attention auto-encoder that constructs neighborhood graphs from spatial coordinates and performs self-supervised multi-omics integration) — https://github.com/Linliu-Bioinf/SSGATE
- **SCANPY** (Leiden clustering of the integrated embedding and UMAP/spatial visualization)
- **ClusterProfiler** (GO enrichment analysis on high-confidence DEGs to annotate cluster functions)
- **MOFA** (Competitive/alternative unsupervised multi-omics integration method referenced for comparison)
- **Seurat** (Network-based competitive integration method (weighted nearest neighbor graph) referenced for comparison)

## Examples

```
adata = Cal_Nbrs_Net(adata, feat='spatial', k_cutoff=15, model='KNN'); adata = prune_net(adata); adata_st, adata_sp = ssgate.train(adata_st, adata_sp, hidden_dims1=128, hidden_dims2=128, out_dims=30, n_epochs=300, lr=0.001, sigma=0.1)
```

## Evaluation signals

- UMAP of the 30-dim embedding shows well-separated clusters
- Spatial cluster map shows strong spatial aggregation with clear boundaries consistent with the outer cortex and inner medulla of the mouse thymus (qualitative check; SCS_MT has no ground-truth labels)
- Neighborhood network correctly stored in adata.uns['nbrs_net'] after Cal_Nbrs_Net and prune_net, with pruned edges only connecting cells in the same Leiden cluster
- Integrated embedding dimensionality equals out_dims=30
- Per-cluster DEGs yield non-trivial GO enrichment terms consistent with annotated cluster functions

## Limitations

- Hyperparameters were set 'based on experience' as defaults rather than systematically derived
- Integration of omics beyond transcriptome/proteome requires normalization and dimensionality-reduction methods that are not yet available (noted as future work)
- No software versions are reported for the compared methods (e.g., scMM, totalVI, MOFA+)
- SCS_MT lacks ground-truth labels, so validation is a qualitative spatial-aggregation check rather than quantitative accuracy

## Evidence

- [abstract] Existing methods are mainly designed for single-cell multi-omics data with little consideration on spatial information, and still have the room for performance improvement.: "Existing methods are mainly designed for single-cell multi-omics data with little consideration on spatial information"
- [readme] The spatial coordinates from spatial multi-omics data are required to be stored in anndata.obsm['spatial']: "The spatial coordinates from spatial multi-omics data are required to be stored in `anndata.obsm['spatial']`"
- [readme] This function is used to prune the adjacency network... ensuring that each edge only connects cells within the same cell population.: "This function is used to prune the adjacency network calculated by the `Cal_Nbrs_Net` function, ensuring that each edge 
only connects cells within the
same cell population."
- [abstract] SSGATE shows better performance and stronger robustness than competitive methods and facilitates downstream analysis.: "SSGATE shows better performance and stronger robustness than competitive methods and facilitates downstream analysis."
- [discussion] Based on experience, we provide a set of hyperparameters suitable for most datasets as default settings: "Based on experience, we provide a set of hyperparameters suitable for most datasets as default settings"
- [discussion] the integration of other single-omics data requires reasonable normalization and dimensionality reduction methods: "the integration of other single-omics data requires reasonable normalization and dimensionality reduction methods, which is also the focus of our subsequent research."
