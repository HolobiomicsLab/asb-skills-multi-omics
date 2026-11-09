---
name: snf-similarity-network-fusion
description: Use when integrating multiple scRNAseq clustering assignments into one
  consensus labeling via Similarity Network Fusion, especially when per-method clusterings
  disagree and ARI vs ground truth must be maximized.
when_to_use_negative:
- You only want the simple element-wise average of the binary matrices — use CHAI-AvgSim
  instead of SNF fusion
- The input is raw expression data and no per-algorithm clustering assignments (or
  binary similarity matrices) have been generated yet
- You need the SAME-clustering EM mixture-model ensemble baseline, which uses SC3/CIDR/Seurat/t-SNE+k-means/SIMLR
  rather than SNF
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0099
tools:
- name: CHAI-SNF
  role: Wrapper function (CHAI_SNF) that fuses binary similarity matrices via SNF
    and runs Spectral Clustering
  repo: https://github.com/lodimk2/chai
- name: SNFtool
  role: Provides the SNF algorithm, run with default parameters, to fuse the seven
    binary similarity matrices
- name: CHAI
  role: R package supplying the seven constituent clustering algorithms and binary
    similarity matrix construction
  repo: https://github.com/lodimk2/chai
provenance:
  source_task_ids:
  - task_002
  - task_001
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:62684fd70e465625472f7645ae8356a6c873e14d2cbe0e7ff5093410835a3359
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
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# snf-similarity-network-fusion

## Summary

CHAI-SNF aggregates the seven per-algorithm binary similarity matrices from scRNAseq clustering (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC) via Similarity Network Fusion (SNFtool) and runs Spectral Clustering on the fused snf_matrix to produce consensus cell-type assignments.

## When to use

Use when integrating multiple scRNAseq clustering assignments into one consensus labeling via Similarity Network Fusion, especially when per-method clusterings disagree and ARI vs ground truth must be maximized.

## When NOT to use

- You only want the simple element-wise average of the binary matrices — use CHAI-AvgSim instead of SNF fusion
- The input is raw expression data and no per-algorithm clustering assignments (or binary similarity matrices) have been generated yet
- You need the SAME-clustering EM mixture-model ensemble baseline, which uses SC3/CIDR/Seurat/t-SNE+k-means/SIMLR rather than SNF

## Inputs

- Seven m x m binary similarity matrices (same cluster = 1, different cluster = 0), one per constituent algorithm (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC)
- SingleCellExperiment object with counts and logcounts assays (e.g. Baron Mouse 1 example data)
- Best_k value for Spectral Clustering (or eval=TRUE to select it by silhouette score)

## Outputs

- snf_matrix: an m x m cell-to-cell similarity matrix with entries in [0,1] (snf_matrix.csv)
- Per-cell consensus cluster assignments (consensus_assignments.csv)
- ARI/NMI vs ground-truth labels for benchmarking

## How to apply

First run the seven constituent clustering algorithms (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC) on the SingleCellExperiment object (counts plus logcounts, e.g. via scuttle::logNormCounts) and convert each clustering assignment into an m x m binary similarity matrix where cell pairs sharing a cluster get 1 and all others 0. Apply the SNF algorithm (SNFtool R package, default parameters) across all seven binary similarity matrices, iteratively fusing the cell-to-cell similarity networks via message passing until convergence to yield snf_matrix. Determine best_k by evaluating the average silhouette score for candidate k in the range 2 to k+1 (k being the true number of clusters when known, as in the paper's benchmarking) — best_k is evaluated separately for the SNF matrix because its entry distribution differs from the AvgSim matrix. Run Spectral Clustering on snf_matrix with best_k to obtain the final consensus cluster assignments. In the CHAI package this whole path is wrapped by the CHAI_SNF(sce, best_k, eval = TRUE) function; the paper reports CHAI-SNF as robust to dataset size/complexity changes on Zheng68K subsamples.

## Related tools

- **CHAI-SNF** (Wrapper function (CHAI_SNF) that fuses binary similarity matrices via SNF and runs Spectral Clustering) — https://github.com/lodimk2/chai
- **SNFtool** (Provides the SNF algorithm, run with default parameters, to fuse the seven binary similarity matrices)
- **CHAI** (R package supplying the seven constituent clustering algorithms and binary similarity matrix construction) — https://github.com/lodimk2/chai

## Examples

```
devtools::install_github("lodimk2/chai"); library(chai); best_k <- 15; sce <- CHAI_SNF(sce, best_k, eval = TRUE)
```

## Evaluation signals

- snf_matrix is an m x m cell-to-cell similarity matrix with all entries in [0,1]
- Every cell receives exactly one consensus cluster label
- The chosen best_k corresponds to the maximum average silhouette score over the evaluated range (2 to k+1)
- CHAI-SNF's ARI against ground truth exceeds the individual algorithms and SAME-clustering on benchmark datasets (Baron, Muraro, SC-Mixology, Zeisel, Zheng68K)
- On Zheng68K subsamples, CHAI-SNF shows the smallest simple-vs-challenging percent difference in ARI, reproducing reported robustness to cluster-complexity changes

## Limitations

- An immediate limitation of CHAI is that it is not currently possible to select an ideal set of algorithms to be used in the final consensus; the recommendation is to include as many algorithms as possible
- Individual algorithms show large variation in performance — e.g. Spectrum often displays subzero and negative ARI — which degrades the fused SNF consensus when included
- best_k evaluation for the SNF matrix must be done separately because the snf_matrix distribution differs from the AvgSim matrix distribution

## Evidence

- [methods] CHAI-SNF module generates its integrated consensus matrix by applying the SNF function (create_snf_matrix) to the list of binary similarity matrices: "we apply the SNF algorithm across all binary similarity matrices [21]"
- [intro] CHAI-AvgSim vs CHAI-SNF aggregation routes: "CHAI-AvgSim is performed by aggregating all clustering assignments with an average similarity matrix, and performing Spectral Clustering"
- [methods] Binary similarity matrix construction rule: "If two cells have the same clustering assignment, assign a value of 1 to a binary similarity matrix corresponding to the two cells."
- [methods] best_k selection by silhouette evaluation: [evidence span withheld — anchor status: not_found]
- [readme] README invocation of the SNF wrapper with eval flag: "# If "eval" is set to TRUE, CHAI will evaluate the best_k for Spectral Clustering using silhouette score."
- [discussion] Algorithm-selection limitation: "An immediate limitation of CHAI is that it is not currently possible to select an ideal set of algorithms to be used in the final consensus"
