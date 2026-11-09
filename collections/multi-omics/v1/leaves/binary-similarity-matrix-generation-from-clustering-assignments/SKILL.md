---
name: binary-similarity-matrix-generation-from-clustering-assignments
description: Use when you have cluster labels per cell from one or more scRNAseq clustering
  algorithms (e.g. Seurat-Louvain, SC3, Spectrum) and need per-algorithm similarity
  matrices as input to CHAI consensus aggregation.
when_to_use_negative:
- When clustering results are probability/membership vectors rather than hard assignments
  — CHAI's binary matrices require discrete same-cluster/different-cluster labels
- When you need weighted similarity from expression profiles directly — this skill
  deliberately binarizes assignments and discards continuous similarity information
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CHAI
  role: R package that runs the seven component clustering algorithms and constructs
    binary similarity matrices from each assignment
  repo: https://github.com/lodimk2/chai
- name: SNFtool
  role: Performs Similarity Network Fusion across the binary similarity matrices to
    produce the CHAI-SNF integrated matrix
- name: GraphST
  role: Source of the spatial KNN binary adjacency representation (K=3) adopted by
    CHAI-ST for spatial transcriptomics integration
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:c2bad98c4e529dbf004674d12db15cb259f25a76e25ec3f30c3dbd7fd4b44466
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# binary-similarity-matrix-generation-from-clustering-assignments

## Summary

Converts each cell-level clustering assignment into a binary similarity matrix (same cluster = 1, different cluster = 0) so that heterogeneous clustering results can be aggregated via CHAI-AvgSim or CHAI-SNF consensus clustering.

## When to use

Use when you have cluster labels per cell from one or more scRNAseq clustering algorithms (e.g. Seurat-Louvain, SC3, Spectrum) and need per-algorithm similarity matrices as input to CHAI consensus aggregation.

## When NOT to use

- When clustering results are probability/membership vectors rather than hard assignments — CHAI's binary matrices require discrete same-cluster/different-cluster labels
- When you need weighted similarity from expression profiles directly — this skill deliberately binarizes assignments and discards continuous similarity information

## Inputs

- Clustering assignment vectors (one per cell) from each of the seven CHAI component algorithms (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC)
- SingleCellExperiment object with counts and logcounts assays (e.g. baron_mouse_1)
- For spatial extension: spatial coordinates of spots (e.g. SCP2046, SCP2331 datasets) for K=3 KNN graph construction

## Outputs

- One binary similarity matrix per clustering algorithm (cell x cell, entries 1 if same cluster else 0)
- Optional spatial binary adjacency matrix (KNN, K=3) for CHAI-ST integration
- Aggregated Average Similarity matrix (CHAI-AvgSim) or fused SNF matrix (CHAI-SNF) ready for Spectral Clustering

## How to apply

Run each of CHAI's seven component clustering algorithms (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC) on the SingleCellExperiment object, then for each algorithm build a binary similarity matrix over cell pairs: assign 1 if two cells share the same cluster assignment and 0 otherwise. These matrices become the common representation that CHAI-AvgSim aggregates by averaging and CHAI-SNF fuses via the SNF algorithm (SNFtool R package, default parameters). The same construction is used to encode non-transcriptomic data: for spatial transcriptomics (e.g. Vandenbon Liver Cancer SCP2046, Savas Breast Cancer SCP2331), compute pairwise spot distances, build an undirected KNN graph with K=3, and encode neighbors as 1 and all other pairs as 0 following the GraphST representation adopted by CHAI-ST. The final integrated matrix (snf_matrix or AvgSim_matrix) feeds Spectral Clustering with best_k chosen by maximum silhouette score in the range 2 to k+1.

## Related tools

- **CHAI** (R package that runs the seven component clustering algorithms and constructs binary similarity matrices from each assignment) — https://github.com/lodimk2/chai
- **SNFtool** (Performs Similarity Network Fusion across the binary similarity matrices to produce the CHAI-SNF integrated matrix)
- **GraphST** (Source of the spatial KNN binary adjacency representation (K=3) adopted by CHAI-ST for spatial transcriptomics integration)

## Examples

```
sce <- SingleCellExperiment(assays = list(counts = as.matrix(baron_mouse_1))); sce <- scuttle::logNormCounts(sce); sce <- CHAI_SNF(sce, best_k = 15, eval = TRUE)
```

## Evaluation signals

- Each binary matrix must be square (cells x cells) with values only in {0, 1}
- Matrix must be symmetric: if cell i and j are co-clustered, entries [i,j] and [j,i] both equal 1
- Diagonal entries should be 1 (every cell is in the same cluster as itself)
- Number of binary matrices produced must equal the number of component algorithms run (seven for the full CHAI pipeline)
- After fusion, Spectral Clustering ARI against ground-truth labels must fall within [-1, 1]; for CHAI-ST, integrated ARI should exceed baseline CHAI-SNF ARI on both spatial datasets

## Limitations

- Spectrum often displays subzero and negative ARI, so its binary matrix may contribute poor-quality signal to the consensus
- Individual clustering algorithms demonstrate large variation in performance, and CHAI currently offers no method to select an ideal subset of algorithms for the consensus
- The binary encoding discards distance/strength information — only same-cluster vs different-cluster membership is retained

## Evidence

- [intro] Create binary similarity matrices from each clustering assignment (same cluster = 1, different cluster = 0): "If two cells are assigned the same cluster, assign a score of 1."
- [methods] Methods description of per-algorithm binary matrix construction: "If two cells have the same clustering assignment, assign a value of 1 to a binary similarity matrix corresponding to the two cells."
- [methods] SNF aggregation of binary similarity matrices: "we apply the SNF algorithm across all binary similarity matrices [21]"
- [readme] README showing CHAI-SNF wrapper and silhouette-based best_k evaluation: "If "eval" is set to TRUE, CHAI will evaluate the best_k for Spectral Clustering using silhouette score."
- [abstract] Spatial transcriptomics integration finding: "we demonstrate CHAI-SNF to have improved performance when including spatial transcriptomics data"
