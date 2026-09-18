---
name: spectral-clustering-on-similarity-matrix
description: Use when you have per-algorithm clustering assignments for scRNAseq cells
  (e.g. from Seurat-Louvain, SC3, Spectrum, scSHC, CHOIR, RACEID, Seurat-SLC) and
  need to fuse them into a single consensus cluster label per cell.
when_to_use_negative:
- When you need a method to select the ideal subset of component clustering algorithms
  for the consensus — the article notes this is an immediate limitation of CHAI and
  recommends, pending community best practices, including as many algorithms as possible.
- When the input is raw expression data with no per-algorithm cluster assignments
  — the spectral clustering step operates on integrated similarity matrices, not on
  counts directly.
- When relying on SAME-clustering as the ensemble alternative and SC3 must run — SC3
  does not run in the SAME-clustering package due to a bug, so comparisons in that
  setting are not reproducible.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: CHAI
  role: R package wrapping the entire consensus pipeline; its CHAI_AvgSim and CHAI_SNF
    functions run Spectral Clustering on the integrated matrix with optional silhouette-based
    best_k evaluation
  repo: https://github.com/lodimk2/chai
- name: CHAI-AvgSim
  role: Aggregates the binary similarity matrices into the Average Similarity matrix
    on which Spectral Clustering is run
  repo: https://github.com/lodimk2/chai
- name: CHAI-SNF
  role: Fuses the binary similarity matrices via SNF (create_snf_matrix) and runs
    Spectral Clustering on the resulting snf_matrix; best_k is evaluated separately
    for this matrix distribution
  repo: https://github.com/lodimk2/chai
- name: SNFtool
  role: Provides the SNF algorithm used to fuse the seven binary similarity matrices
    into snf_matrix before spectral clustering
merged_aliases:
- spectral-clustering-on-fused-similarity-matrix
merged_alias_records:
- alias: spectral-clustering-on-fused-similarity-matrix
  slug: spectral-clustering-on-fused-similarity-matrix
  jaccard_score: 0.8
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:88975a182e6ecbb889cd31baa96a2d5f36ca460eccd45f1d1e23dee82eed0b65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lodimk2/chai
  tools:
  - CHAI
  - CHAI-AvgSim
  - CHAI-SNF
  - R
  - SC3
  - SNFtool
  - Seurat
  - Spectrum
  - scSHC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# spectral-clustering-on-similarity-matrix

## Summary

Final consensus step of the CHAI framework: Spectral Clustering is run on an integrated cell-to-cell similarity matrix (either the Average Similarity matrix from CHAI-AvgSim or the SNF-fused matrix from CHAI-SNF) to convert pairwise co-clustering similarities into discrete consensus cell-type assignments.

## When to use

Use when you have per-algorithm clustering assignments for scRNAseq cells (e.g. from Seurat-Louvain, SC3, Spectrum, scSHC, CHOIR, RACEID, Seurat-SLC) and need to fuse them into a single consensus cluster label per cell.

## When NOT to use

- When you need a method to select the ideal subset of component clustering algorithms for the consensus — the article notes this is an immediate limitation of CHAI and recommends, pending community best practices, including as many algorithms as possible.
- When the input is raw expression data with no per-algorithm cluster assignments — the spectral clustering step operates on integrated similarity matrices, not on counts directly.
- When relying on SAME-clustering as the ensemble alternative and SC3 must run — SC3 does not run in the SAME-clustering package due to a bug, so comparisons in that setting are not reproducible.

## Inputs

- AvgSim_matrix (m x m average of binary similarity matrices) or snf_matrix (SNF-fused similarity matrix with entries in [0,1])
- Seven binary similarity matrices, one per clustering algorithm (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC)
- best_k (candidate number of clusters; silhouette evaluation searches range 2 to k+1)
- SingleCellExperiment object with counts and logcounts assays (as used by the CHAI wrappers)

## Outputs

- Per-cell consensus cluster assignments (e.g. cluster_assignments.csv with exactly one label per cell, or consensus_assignments.csv)
- Fused SNF matrix (snf_matrix.csv) when the CHAI-SNF path is used
- Selected best_k corresponding to the maximum average silhouette score over the evaluated range

## How to apply

First convert each of the seven per-algorithm clustering assignments into a binary similarity matrix, assigning 1 if two cells share a cluster and 0 otherwise, then aggregate into either the AvgSim_matrix (average across algorithms) or the snf_matrix (SNFtool fusion via message passing). If the true number of clusters is unknown, determine best_k by evaluating the average silhouette score over candidate k values in the range 2 to k+1, where k is the true number of clusters when known (best_k is evaluated separately for the SNF matrix because its distribution differs from the AvgSim matrix). Run Spectral Clustering on the chosen integrated matrix with best_k to obtain final cell identities, then write per-cell assignments (e.g. cluster_assignments.csv / consensus_assignments.csv). In the CHAI R package this is wrapped by CHAI_AvgSim(sce, best_k, eval = TRUE) or CHAI_SNF(sce, best_k, eval = TRUE), where eval = TRUE triggers silhouette-based best_k evaluation on the SingleCellExperiment object.

## Related tools

- **CHAI** (R package wrapping the entire consensus pipeline; its CHAI_AvgSim and CHAI_SNF functions run Spectral Clustering on the integrated matrix with optional silhouette-based best_k evaluation) — https://github.com/lodimk2/chai
- **CHAI-AvgSim** (Aggregates the binary similarity matrices into the Average Similarity matrix on which Spectral Clustering is run) — https://github.com/lodimk2/chai
- **CHAI-SNF** (Fuses the binary similarity matrices via SNF (create_snf_matrix) and runs Spectral Clustering on the resulting snf_matrix; best_k is evaluated separately for this matrix distribution) — https://github.com/lodimk2/chai
- **SNFtool** (Provides the SNF algorithm used to fuse the seven binary similarity matrices into snf_matrix before spectral clustering)

## Examples

```
library(chai); best_k <- 15; sce <- CHAI_AvgSim(sce, best_k, eval = TRUE)
```

## Evaluation signals

- The integrated matrix is an m x m cell-to-cell similarity matrix with entries in [0,1] (for the SNF path).
- cluster_assignments.csv contains exactly one cluster label for every cell (e.g. all 5759 Savas breast cancer CD3+ cells).
- The chosen best_k corresponds to the maximum average silhouette score over the evaluated range 2 to k+1.
- Downstream validation: FindAllMarkers on the CHAI clusters yields a marker_summary.csv in which CDH3 is significantly upregulated in exactly one cluster (leader tumor cell cluster on the Savas dataset).
- Silhouette-based best_k evaluation is performed separately for the SNF matrix versus the AvgSim matrix, since their similarity distributions differ.

## Limitations

- No method exists to select an ideal set of algorithms for the final consensus; component algorithms vary widely in performance (e.g. Spectrum often shows subzero and negative ARI), and the current recommendation is to include as many algorithms as possible.
- best_k evaluation covers range 2 to k+1, with k being the true number of clusters — the range is not fully determined when the true cluster count is unknown.
- Further evaluation remains to be done on which algorithms are best for particular dataset conditions.

## Evidence

- [intro] CHAI-AvgSim is performed by aggregating all clustering assignments with an average similarity matrix, and performing Spectral Clustering: "CHAI-AvgSim is performed by aggregating all clustering assignments with an average similarity matrix, and performing Spectral Clustering"
- [methods] we conduct a silhouette score evaluation in range 2 to k + 1, with k being the true number of clusters present in the dataset: [evidence span withheld — anchor status: not_found]
- [methods] spectral clustering is then run on this snf_matrix, and best_k is evaluated separately for the SNF matrix because its distribution differs from the AvgSim matrix: [evidence span withheld — anchor status: not_found]
- [readme] If "eval" is set to TRUE, CHAI will evaluate the best_k for Spectral Clustering using silhouette score.: "If "eval" is set to TRUE, CHAI will evaluate the best_k for Spectral Clustering using silhouette score."
