---
name: single-cell-differential-expression-analysis
description: Use when you have per-cell cluster assignments (e.g. from CHAI-AvgSim
  or CHAI-SNF spectral clustering) and need marker genes per cluster to biologically
  annotate clusters, such as finding the CDH3-enriched tumor cell cluster.
when_to_use_negative:
- You do not yet have cluster assignments — run the CHAI consensus clustering (component
  algorithms, AvgSim/SNF aggregation, Spectral Clustering) first before marker analysis.
- The goal is to decide the number of clusters (best_k) — that is determined by silhouette
  score evaluation in CHAI, not by differential expression.
- Input is bulk RNA-seq or spatial-only data without per-cell clustering; FindAllMarkers
  per-cluster DE requires single-cell cluster groupings.
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: Provides FindAllMarkers, used to compute per-cluster differentially expressed
    genes on the CHAI consensus clusters
- name: CHAI
  role: Produces the consensus cluster assignments (cluster_assignments.csv) on which
    differential expression is performed
  repo: https://github.com/lodimk2/chai
- name: R
  role: Language in which CHAI (open source R package) and the Seurat marker workflow
    are executed
  repo: https://github.com/lodimk2/chai
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:c7573a8ca97bf5f38bc1fc7cefb97d0bdbc839519eba59921fa723c11cf13d97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lodimk2/chai
  tools:
  - CHAI
  - CHAI-AvgSim
  - R
  - SC3
  - Seurat
  - Spectrum
  - scSHC
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# single-cell-differential-expression-analysis

## Summary

Per-cluster differential expression on consensus (CHAI) cluster assignments using Seurat's FindAllMarkers, to identify marker genes that label each cell cluster — e.g. flagging the tumor leader cell cluster in the Savas breast cancer TNBC dataset by CDH3 enrichment.

## When to use

Use when you have per-cell cluster assignments (e.g. from CHAI-AvgSim or CHAI-SNF spectral clustering) and need marker genes per cluster to biologically annotate clusters, such as finding the CDH3-enriched tumor cell cluster.

## When NOT to use

- You do not yet have cluster assignments — run the CHAI consensus clustering (component algorithms, AvgSim/SNF aggregation, Spectral Clustering) first before marker analysis.
- The goal is to decide the number of clusters (best_k) — that is determined by silhouette score evaluation in CHAI, not by differential expression.
- Input is bulk RNA-seq or spatial-only data without per-cell clustering; FindAllMarkers per-cluster DE requires single-cell cluster groupings.

## Inputs

- Per-cell consensus cluster assignments (cluster_assignments.csv, one assignment per cell, e.g. 5759 cells from Savas Breast Cancer SCP2331)
- Single-cell RNA-seq count matrix as a Seurat object / SingleCellExperiment with counts (and logcounts) assay

## Outputs

- marker_summary.csv — per-cluster differentially expressed genes from FindAllMarkers (gene, cluster, log fold change, significance)
- Named cluster label, e.g. the CDH3-enriched tumor cell cluster ID

## How to apply

Take the consensus cluster assignments (one label per cell, e.g. from CHAI spectral clustering on the AvgSim_matrix) into a Seurat object. Run Seurat's FindAllMarkers on the CHAI clusters to compute per-cluster differentially expressed genes relative to all other clusters, writing the marker table (cluster, gene, avg_logFC, p-value) to marker_summary.csv. Scan the marker summary for the gene of interest — in the Savas Breast Cancer (TNBC) CD3+ dataset (SCP2331, 5759 cells) the target gene is CDH3 — and confirm it is significantly upregulated in exactly one cluster; report that cluster as the tumor (leader) cell cluster. Validation requires marker_summary.csv to show CDH3 significantly upregulated in exactly one CHAI cluster and cluster_assignments.csv to contain one assignment per cell for all 5759 cells.

## Related tools

- **Seurat** (Provides FindAllMarkers, used to compute per-cluster differentially expressed genes on the CHAI consensus clusters)
- **CHAI** (Produces the consensus cluster assignments (cluster_assignments.csv) on which differential expression is performed) — https://github.com/lodimk2/chai
- **R** (Language in which CHAI (open source R package) and the Seurat marker workflow are executed) — https://github.com/lodimk2/chai

## Examples

```
sce <- CHAI_AvgSim(sce, best_k, eval = TRUE); markers <- Seurat::FindAllMarkers(seurat_obj, only.pos = TRUE); write.csv(markers, "marker_summary.csv")
```

## Evaluation signals

- marker_summary.csv shows CDH3 significantly upregulated in exactly one CHAI cluster relative to all other clusters
- cluster_assignments.csv contains exactly one assignment per cell for all 5759 cells of the Savas dataset
- The identified cluster is consistent with the article's finding of a leader tumor cell cluster enriched with CDH3
- Marker table includes standard FindAllMarkers columns (gene, cluster, average log fold change, p-value) for every CHAI cluster

## Limitations

- Marker-based cluster identification depends on the upstream consensus clustering; the article notes individual clustering algorithms demonstrate large variation in performance, which propagates to cluster labels.
- The paper recommends including as many algorithms as possible in the CHAI consensus (pending community best practices), so down-selected pipelines may change the clusters and thus the marker results.
- This workflow was demonstrated on the Savas Breast Cancer dataset (SCP2331, 5759 cells); behavior on other tissues or marker genes must be re-validated.

## Evidence

- [other] Use Seurat's FindAllMarkers function on the CHAI clusters to compute per-cluster differentially expressed genes: [evidence span withheld — anchor status: not_found]
- [other] Identify the cluster in which CDH3 is the top enriched marker and report it as the tumor cell cluster: [evidence span withheld — anchor status: not_found]
- [other] the output is acceptable if marker_summary.csv shows CDH3 significantly upregulated in exactly one CHAI cluster: [evidence span withheld — anchor status: not_found]
- [abstract] We demonstrate CHAI's practical use case by identifying a leader tumor cell cluster enriched with CDH3: "We demonstrate CHAI's practical use case by identifying a leader tumor cell cluster enriched with CDH3"
- [readme] CHAI is available as an open source R package on GitHub: [evidence span withheld — anchor status: not_found]
