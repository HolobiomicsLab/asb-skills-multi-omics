---
name: similarity-network-fusion-of-binary-matrices
description: Use when you have cluster assignments (or an extra omics similarity matrix,
  e.g. spatial-coordinate KNN) from multiple scRNAseq clustering methods and need
  a single fused similarity matrix for consensus cell-type identification.
when_to_use_negative:
- When you only need a simple aggregate of cluster agreements without network fusion
  — CHAI-AvgSim (average similarity matrix) may suffice
- When input is raw counts with no per-algorithm cluster assignments yet; component
  algorithms must be run first to produce binary similarity matrices
- When a single clustering method is known to be optimal for the dataset — component
  algorithms show large variation in performance and the consensus adds no benefit
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CHAI
  role: R package that runs the seven component clustering algorithms, builds binary
    similarity matrices, and applies SNF to produce the snf_matrix
  repo: https://github.com/lodimk2/chai
- name: SNFtool
  role: R package implementing the Similarity Network Fusion algorithm applied across
    the binary similarity matrices (Wang et al. 2014)
- name: R
  role: Language in which CHAI is implemented and distributed as an open source package
- name: GraphST
  role: Source of the spatial-representation style used to build the K=3 KNN binary
    spatial matrix integrated via CHAI-ST
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:454b73a50bff6be4d4efaf54ef72a83856e4b0ecffa31bdb98494ef4069eb43c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lodimk2/chai
  tools:
  - CHAI
  - CHAI-SNF
  - GraphST
  - R
  - SNFtool
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# similarity-network-fusion-of-binary-matrices

## Summary

Aggregates per-algorithm binary similarity matrices (same cluster = 1, different cluster = 0) from CHAI's seven scRNAseq clustering methods into a fused consensus network via the SNF algorithm (SNFtool/Wang et al. 2014), optionally including an additional omics matrix such as a GraphST-derived spatial KNN binary matrix, before Spectral Clustering.

## When to use

Use when you have cluster assignments (or an extra omics similarity matrix, e.g. spatial-coordinate KNN) from multiple scRNAseq clustering methods and need a single fused similarity matrix for consensus cell-type identification.

## When NOT to use

- When you only need a simple aggregate of cluster agreements without network fusion — CHAI-AvgSim (average similarity matrix) may suffice
- When input is raw counts with no per-algorithm cluster assignments yet; component algorithms must be run first to produce binary similarity matrices
- When a single clustering method is known to be optimal for the dataset — component algorithms show large variation in performance and the consensus adds no benefit

## Inputs

- SingleCellExperiment object with counts and logcounts assays (e.g. baron_mouse_1)
- Clustering assignments from the seven CHAI component algorithms (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC)
- Optional spatial transcriptomics coordinates (e.g. Vandenbon Liver Cancer SCP2046, Savas Breast Cancer SCP2331) for building a K=3 KNN binary adjacency matrix
- best_k value or true number of clusters for Spectral Clustering

## Outputs

- Fused snf_matrix (consensus similarity network)
- Consensus cell-type cluster assignments from Spectral Clustering on the snf_matrix
- ARI against ground-truth labels (e.g. ari_spatial_comparison.csv comparing baseline CHAI-SNF vs CHAI-SNF with spatial matrix)

## How to apply

First run each of CHAI's seven component algorithms (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC) and convert each clustering assignment into a binary similarity matrix where cells sharing a cluster get a value of 1. Optionally append additional binary matrices, e.g. an undirected K=3 nearest-neighbor graph built from spatial coordinates following the GraphST representation used in CHAI-ST. Apply the SNF algorithm across all binary similarity matrices — either CHAI-SNF-First-Level, fusing all matrices including the spatial one in a single SNF run via the SNFtool R package with default parameters, or the baseline CHAI-SNF on the seven algorithm matrices alone. Run Spectral Clustering on the resulting snf_matrix to get final cell identities, selecting best_k by highest silhouette score in the range 2 to k+1 when the true number of clusters is unknown, then score with ARI against ground-truth labels.

## Related tools

- **CHAI** (R package that runs the seven component clustering algorithms, builds binary similarity matrices, and applies SNF to produce the snf_matrix) — https://github.com/lodimk2/chai
- **SNFtool** (R package implementing the Similarity Network Fusion algorithm applied across the binary similarity matrices (Wang et al. 2014))
- **R** (Language in which CHAI is implemented and distributed as an open source package)
- **GraphST** (Source of the spatial-representation style used to build the K=3 KNN binary spatial matrix integrated via CHAI-ST)

## Examples

```
sce <- CHAI_SNF(sce, best_k, eval = TRUE)
```

## Evaluation signals

- All binary similarity matrices contain only 0/1 entries (same cluster = 1, different cluster = 0)
- SNF output is a valid fused similarity matrix on the same cells; Spectral Clustering runs on it without error
- ARI against ground-truth labels falls within the [-1, 1] range
- With spatial integration (CHAI-ST / CHAI-SNF-First-Level), ARI should improve over baseline CHAI-SNF on both the Vandenbon Liver Cancer (SCP2046) and Savas Breast Cancer (SCP2331) datasets
- best_k selection by silhouette score is performed in the range 2 to k+1 when the true cluster number is unknown

## Limitations

- CHAI does not currently provide a way to select an ideal set of algorithms for the final consensus; the recommendation is to include as many algorithms as possible
- Spectrum often displays subzero and negative ARI and is a poor-performing component that can drag down the fusion
- Individual clustering algorithms demonstrate large variation in performance, so consensus quality depends heavily on component quality

## Evidence

- [methods] SNF across binary matrices: "we apply the SNF algorithm across all binary similarity matrices [21]"
- [methods] binary similarity construction: "If two cells have the same clustering assignment, assign a value of 1 to a binary similarity matrix corresponding to the two cells."
- [abstract] spatial improvement finding: "we demonstrate CHAI-SNF to have improved performance when including spatial transcriptomics data"
- [readme] CHAI offers SNF consensus: "is a consensus clustering framework that offers two methods for consensus clustering: Average Similarity (AvgSim) and Similarity Network Fusion (SNF)"
- [methods] silhouette-based k selection: [evidence span withheld — anchor status: not_found]
