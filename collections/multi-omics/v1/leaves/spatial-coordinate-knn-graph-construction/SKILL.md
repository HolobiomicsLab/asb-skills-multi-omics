---
name: spatial-coordinate-knn-graph-construction
description: Use when a SingleCellExperiment or coordinate table includes spatial
  x/Y positions and the goal is to inject spatial proximity as an additional binary
  similarity matrix into CHAI-SNF to improve ARI of cell-type clustering.
when_to_use_negative:
- Data are non-spatial scRNAseq (no x/Y coordinates available), so no GraphST-style
  spatial similarity matrix can be formed — run baseline CHAI-SNF or CHAI-AvgSim instead
- The goal is comparison against GraphST embeddings rather than CHAI-ST-style first-level
  SNF integration; the paper's Savas Breast Cancer benchmark shows CHAI-SNF-ST does
  not always outperform GraphST
- The number of clusters k is unknown and silhouette-based best_k selection in the
  range 2 to k+1 cannot be applied because no ground-truth or expectation of cluster
  count exists
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CHAI
  role: Consensus clustering framework consuming the spatial binary similarity matrix
    alongside per-algorithm binary matrices
  repo: https://github.com/lodimk2/chai
- name: CHAI-SNF
  role: SNF-based aggregation mode that fuses the spatial KNN binary matrix with component-cluster
    binary matrices before Spectral Clustering
  repo: https://github.com/lodimk2/chai
- name: SNFtool
  role: R package applying Similarity Network Fusion with default parameters across
    all binary similarity matrices
- name: GraphST
  role: 'Source of the spatial representation being reproduced: a KNN graph on spatial
    coordinates encoded as a binary X/Y similarity matrix'
- name: R
  role: Implementation language for the CHAI package and the SNF/spectral-clustering
    workflow
  repo: https://github.com/lodimk2/chai
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:eeb587dca3c42e6a914b2536254001b502acab08482dbb516dc6fa37725b8709
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# spatial-coordinate-knn-graph-construction

## Summary

Constructs a binary spatial neighborhood similarity matrix from spatial transcriptomics spot/cell coordinates using an undirected KNN graph (K=3, GraphST representation), for integration into the CHAI-SNF consensus clustering pipeline.

## When to use

Use when a SingleCellExperiment or coordinate table includes spatial x/Y positions and the goal is to inject spatial proximity as an additional binary similarity matrix into CHAI-SNF to improve ARI of cell-type clustering.

## When NOT to use

- Data are non-spatial scRNAseq (no x/Y coordinates available), so no GraphST-style spatial similarity matrix can be formed — run baseline CHAI-SNF or CHAI-AvgSim instead
- The goal is comparison against GraphST embeddings rather than CHAI-ST-style first-level SNF integration; the paper's Savas Breast Cancer benchmark shows CHAI-SNF-ST does not always outperform GraphST
- The number of clusters k is unknown and silhouette-based best_k selection in the range 2 to k+1 cannot be applied because no ground-truth or expectation of cluster count exists

## Inputs

- Spatial coordinates (x, Y positions) for each spot/cell, e.g., from SCP2046 (Vandenbon Liver Cancer) or SCP2331 (Savas Breast Cancer)
- Ground-truth spot label annotations for downstream ARI evaluation (optional)
- Binary similarity matrices from the seven CHAI component clustering algorithms (same cluster = 1, different cluster = 0)

## Outputs

- Binary KNN adjacency matrix (n x n, 1 for K=3 spatial neighbors, 0 otherwise)
- CHAI-SNF-First-Level integrated similarity matrix including the spatial matrix
- Consensus cluster assignments and ARI values (e.g., written to ari_spatial_comparison.csv)

## How to apply

Load the spatial coordinates for the dataset (e.g., Vandenbon/SCP2046 Liver Cancer or Savas/SCP2331 Breast Cancer) and compute pairwise Euclidean distances between spots. Build an undirected KNN neighborhood graph with K=3 following the GraphST representation adopted by CHAI-ST: neighbors receive a value of 1 and all other cell-pairs 0, yielding a binary adjacency matrix of the same dimensions as the per-algorithm binary similarity matrices from CHAI's seven component clusterers. Pass this matrix into the CHAI-SNF-First-Level integration, where SNF (via the SNFtool R package, default parameters) is run once over all binary matrices including the spatial one, followed by Spectral Clustering with best_k selected by the highest silhouette score in the range 2 to k+1.

## Related tools

- **CHAI** (Consensus clustering framework consuming the spatial binary similarity matrix alongside per-algorithm binary matrices) — https://github.com/lodimk2/chai
- **CHAI-SNF** (SNF-based aggregation mode that fuses the spatial KNN binary matrix with component-cluster binary matrices before Spectral Clustering) — https://github.com/lodimk2/chai
- **SNFtool** (R package applying Similarity Network Fusion with default parameters across all binary similarity matrices)
- **GraphST** (Source of the spatial representation being reproduced: a KNN graph on spatial coordinates encoded as a binary X/Y similarity matrix)
- **R** (Implementation language for the CHAI package and the SNF/spectral-clustering workflow) — https://github.com/lodimk2/chai

## Examples

```
library(FNN); knn.idx <- get.knn(as.matrix(spatial_coords), k=3); A <- matrix(0, n, n); A[cbind(rep(1:n,3), as.vector(knn.idx$nn.index))] <- 1
```

## Evaluation signals

- The adjacency matrix is binary with exactly 3 ones per row (undirected symmetry gives 6 per row after symmetrization), matching K=3
- ARI computed after CHAI-SNF with the spatial matrix should exceed baseline CHAI-SNF ARI on both SCP2046 and SCP2331, reproducing the paper's reported improvement
- All reported ARI values must fall within the valid [-1, 1] range
- The spatial matrix must have identical dimensions (n x n spots) to the per-algorithm binary similarity matrices so SNF fusion succeeds
- Spectral Clustering k should equal the best_k chosen by the highest silhouette score evaluated in the range 2 to k+1

## Limitations

- Spectrum, one of CHAI's seven component algorithms, often displays subzero and negative ARI, which can propagate into the consensus when included
- CHAI does not currently provide a method to select an ideal set of algorithms for the final consensus; the recommendation is to include as many algorithms as possible
- Improvement from spatial integration is dataset-dependent: it significantly aids performance on the Vandenbon Liver Cancer dataset, but CHAI-SNF-ST does not outperform GraphST on the Savas Breast Cancer dataset

## Evidence

- [methods] we apply the SNF algorithm across all binary similarity matrices: "we apply the SNF algorithm across all binary similarity matrices [21]"
- [methods] If two cells have the same clustering assignment, assign a value of 1 to a binary similarity matrix corresponding to the two cells.: "If two cells have the same clustering assignment, assign a value of 1 to a binary similarity matrix corresponding to the two cells."
- [abstract] we demonstrate CHAI-SNF to have improved performance when including spatial transcriptomics data: "we demonstrate CHAI-SNF to have improved performance when including spatial transcriptomics data"
- [intro] users can determine the "best_k" for Spectral Clustering by evaluating the optimal silhouette score: "users can determine the "best_k" for Spectral Clustering by evaluating the optimal silhouette score"
- [readme] CHAI is available as an open source R package on GitHub: [evidence span withheld — anchor status: not_found]
