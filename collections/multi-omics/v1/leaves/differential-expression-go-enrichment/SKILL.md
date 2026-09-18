---
name: differential-expression-go-enrichment
description: Use when a Leiden (or equivalent) clustering over an integrated embedding
  — e.g. SSGATE's 30-dim joint transcriptome-proteome output — needs functional interpretation
  via per-cluster marker genes and GO term annotation.
when_to_use_negative:
- Input lacks a clustering result or annotation over the embedding — run Leiden on
  the SSGATE embedding first; DEGs on unclustered cells are meaningless
- Single-omics data with complex nonlinear cross-modality structure is the integration
  problem itself — use SSGATE training first, this skill only annotates the resulting
  clusters
- Data is not in anndata format or expression is not stored in anndata.X — the pipeline
  cannot read it correctly per the README
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0121
tools:
- name: SCANPY
  role: Leiden clustering of the SSGATE embedding, Wilcoxon DEG testing per cluster,
    UMAP and spatial visualization
- name: ClusterProfiler
  role: GO enrichment analysis on filtered high-confidence DEGs to annotate cluster
    functions (Fig. 4B)
- name: SSGATE
  role: Produces the 30-dimensional joint transcriptome-proteome embedding and spatial
    cluster map that the DEG/enrichment step annotates
  repo: https://github.com/Linliu-Bioinf/SSGATE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:74e08677250359da0c7c05671742c6283a4ea2c764f3aed6818f00b6f1534464
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
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# differential-expression-go-enrichment

## Summary

Post-clustering functional annotation workflow: identify per-cluster differentially expressed genes from an integrated multi-omics embedding using the Wilcoxon rank-sum test, then run GO enrichment on high-confidence DEGs with ClusterProfiler to assign biological function to each cluster (e.g. cortex vs medulla populations in SSGATE's mouse thymus analysis, Fig. 4B).

## When to use

Use when a Leiden (or equivalent) clustering over an integrated embedding — e.g. SSGATE's 30-dim joint transcriptome-proteome output — needs functional interpretation via per-cluster marker genes and GO term annotation.

## When NOT to use

- Input lacks a clustering result or annotation over the embedding — run Leiden on the SSGATE embedding first; DEGs on unclustered cells are meaningless
- Single-omics data with complex nonlinear cross-modality structure is the integration problem itself — use SSGATE training first, this skill only annotates the resulting clusters
- Data is not in anndata format or expression is not stored in anndata.X — the pipeline cannot read it correctly per the README

## Inputs

- AnnData (.h5ad) object with SSGATE-integrated joint embedding and clustering result in obs/cluster column
- Raw/normalized expression matrix in anndata.X (transcriptome count-depth scaled + log1p, HVG-selected; proteome CLR-normalized)
- Spatial coordinates in anndata.obsm['spatial'] for spatial cluster maps

## Outputs

- Per-cluster DEG tables with Wilcoxon p-values and log fold changes
- GO enrichment result tables per cluster (ClusterProfiler4) for high-confidence DEGs
- Functional annotation of each Leiden cluster (e.g. cortex vs medulla assignment), UMAP and spatial cluster plots (Fig. 4B)

## How to apply

First cluster the integrated embedding with Leiden (SCANPY, sweeping resolution ~0.1–1.0 until spatially aggregated, well-separated clusters emerge) and render both UMAP and spatial maps overlaid on anndata.obsm['spatial'] coordinates. Then run a Wilcoxon rank-sum test per cluster (SCANPY rank_genes_groups) to compute per-cluster DEGs from the expression matrix (anndata.X). Filter DEGs by p-value and log fold change to retain high-confidence markers, and submit them to GO enrichment with ClusterProfiler4 to annotate each cluster's function. Acceptance is qualitative: enriched GO terms should be consistent with the anatomical interpretation of the spatial clusters (e.g. cortical vs medullary functions in the mouse thymus).

## Related tools

- **SCANPY** (Leiden clustering of the SSGATE embedding, Wilcoxon DEG testing per cluster, UMAP and spatial visualization)
- **ClusterProfiler** (GO enrichment analysis on filtered high-confidence DEGs to annotate cluster functions (Fig. 4B))
- **SSGATE** (Produces the 30-dimensional joint transcriptome-proteome embedding and spatial cluster map that the DEG/enrichment step annotates) — https://github.com/Linliu-Bioinf/SSGATE

## Examples

```
sc.tl.rank_genes_groups(adata, 'leiden', method='wilcoxon'); R: enrichGO(gene = sig_degs, OrgDb = org.Mm.eg.db, keyType = 'SYMBOL', ont = 'BP')  # ClusterProfiler4 on filtered DEGs
```

## Evaluation signals

- UMAP shows well-separated clusters across the Leiden resolution range (0.1–1.0)
- Spatial cluster map displays strong spatial aggregation with clear boundaries; for SCS_MT mouse thymus, clusters should align with outer cortex vs inner medulla
- DEG tables contain per-cluster genes with significant Wilcoxon p-values and log fold changes before enrichment filtering
- GO enrichment yields non-empty, biologically coherent term sets per cluster consistent with the tissue anatomy
- Data schema check: expression in adata.X and spatial coordinates in adata.obsm['spatial']

## Limitations

- SCS_MT spatial multi-omics data has no ground-truth cluster labels, so validation of the cortex/medulla consistency is a qualitative acceptance check only
- SSGATE hyperparameters were set 'based on experience' as defaults rather than systematically derived, so downstream cluster quality and DEGs may be sensitive to parameter choices
- Normalization/dimensionality-reduction schemes exist only for transcriptome/proteome integration; other omics modalities are not yet supported and would yield unreliable clusters and annotations

## Evidence

- [methods] Compute per-cluster differentially expressed genes with the Wilcoxon test and run GO enrichment analysis on high-confidence DEGs (p-value and log fold change filters) with ClusterProfiler4 to annotate cluster functions (Fig. 4B): [evidence span withheld — anchor status: not_found]
- [abstract] facilitates downstream analysis: "SSGATE shows better performance and stronger robustness than competitive methods and facilitates downstream analysis."
- [readme] anndata input requirements: "The expression profiles are required to be stored in anndata.X, otherwise the data cannot be read correctly."
- [results] cluster spatial structure validation: [evidence span withheld — anchor status: not_found]
