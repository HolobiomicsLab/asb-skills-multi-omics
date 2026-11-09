---
name: average-similarity-matrix-aggregation
description: Use when you have multiple scRNAseq cluster assignment vectors for the
  same cells and need a single consensus clustering, or when no single clustering
  method performs consistently across your dataset conditions.
when_to_use_negative:
- You need multiomic or spatial transcriptomics integration — the article reports
  CHAI-SNF has improved performance when including spatial transcriptomics data
- You need to select an ideal subset of component algorithms — the article states
  this is not currently possible and recommends including as many algorithms as possible
- Your input is a precomputed distance/network rather than discrete cluster assignments
  per algorithm — the binary-similarity construction assumes assignment labels
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: CHAI
  role: R package providing CHAI_AvgSim wrapper, create_matrix_list, create_average_matrix,
    calc_silhouette_scores, and Spectral Clustering on the AvgSim_matrix
  repo: https://github.com/lodimk2/chai
- name: Seurat
  role: Component clusterer contributing Seurat-Louvain and Seurat-SLC assignments
    to the binary similarity matrices
- name: CHOIR
  role: Component clusterer using statistical significance testing; contributes assignments
    to the AvgSim aggregation
- name: RACEID
  role: Component clusterer contributing assignments to the binary similarity matrix
    list
- name: SC3
  role: Component clusterer (consensus clustering via CSPA) contributing assignments
    to the AvgSim aggregation
- name: Spectrum
  role: Component clusterer contributing assignments to the AvgSim aggregation
- name: scSHC
  role: Component clusterer using statistical significance testing; contributes assignments
    to the AvgSim aggregation
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:d4342a13662eac2285b7dbfb66b441dd9cc76a94da271c96a1479db874acc483
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lodimk2/chai
  tools:
  - CHAI
  - CHAI-AvgSim
  - CHOIR
  - R
  - RACEID
  - SC3
  - Seurat
  - Spectrum
  - scSHC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# average-similarity-matrix-aggregation

## Summary

CHAI-AvgSim consensus clustering: convert each of seven component scRNAseq clustering assignments (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC) into an m x m binary similarity matrix, aggregate them element-wise into an Average Similarity (AvgSim) matrix, and run Spectral Clustering on it to obtain consensus cell identities.

## When to use

Use when you have multiple scRNAseq cluster assignment vectors for the same cells and need a single consensus clustering, or when no single clustering method performs consistently across your dataset conditions.

## When NOT to use

- You need multiomic or spatial transcriptomics integration — the article reports CHAI-SNF has improved performance when including spatial transcriptomics data
- You need to select an ideal subset of component algorithms — the article states this is not currently possible and recommends including as many algorithms as possible
- Your input is a precomputed distance/network rather than discrete cluster assignments per algorithm — the binary-similarity construction assumes assignment labels

## Inputs

- SingleCellExperiment object with counts and logcounts assays
- Per-cell cluster assignment tables from the seven CHAI component algorithms (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC)
- Candidate k value (best_k) or a flag to evaluate best_k via silhouette score

## Outputs

- AvgSim_matrix (m x m element-wise average of binary similarity matrices)
- Per-cell consensus cluster assignments (consensus_assignments.csv)
- Selected best_k with its silhouette score
- Per-cluster marker summary (marker_summary.csv) via Seurat FindAllMarkers

## How to apply

Load a SingleCellExperiment object (counts and logcounts defined, e.g. via scuttle::logNormCounts on a baron_mouse_1-style matrix) and run the seven CHAI component algorithms. Convert each algorithm's per-cell assignments into a binary similarity matrix via create_matrix_list, assigning entry 1 when two cells share the same cluster and 0 otherwise. Aggregate the binary matrices element-wise with create_average_matrix to form the AvgSim_matrix. If the true number of clusters is unknown, compute silhouette scores over the range 2 to k+1 (calc_silhouette_scores) and set best_k to the maximum-silhouette value; otherwise supply best_k directly. Run Spectral Clustering on the AvgSim_matrix with best_k clusters to obtain final consensus cell identities.

## Related tools

- **CHAI** (R package providing CHAI_AvgSim wrapper, create_matrix_list, create_average_matrix, calc_silhouette_scores, and Spectral Clustering on the AvgSim_matrix) — https://github.com/lodimk2/chai
- **Seurat** (Component clusterer contributing Seurat-Louvain and Seurat-SLC assignments to the binary similarity matrices)
- **CHOIR** (Component clusterer using statistical significance testing; contributes assignments to the AvgSim aggregation)
- **RACEID** (Component clusterer contributing assignments to the binary similarity matrix list)
- **SC3** (Component clusterer (consensus clustering via CSPA) contributing assignments to the AvgSim aggregation)
- **Spectrum** (Component clusterer contributing assignments to the AvgSim aggregation)
- **scSHC** (Component clusterer using statistical significance testing; contributes assignments to the AvgSim aggregation)

## Examples

```
sce <- CHAI_AvgSim(sce, best_k, eval = TRUE)
```

## Evaluation signals

- Each cell receives exactly one consensus cluster label (e.g. cluster_assignments.csv contains one assignment per cell for all 5759 Savas cells)
- The recorded best_k attains the maximum silhouette score across the evaluated range 2 to k+1
- Downstream marker analysis (FindAllMarkers) shows the expected marker enriched in exactly one CHAI cluster relative to all others, e.g. CDH3 significantly upregulated in one cluster
- AvgSim_matrix entries fall in [0, 1] since they are element-wise averages of binary 0/1 similarity matrices across all seven algorithms

## Limitations

- Not currently possible to select an ideal set of algorithms for the final consensus; current recommendation is to include as many algorithms as possible
- Individual clustering algorithms demonstrate large variation in performance, and Spectrum often displays subzero and negative ARI (a poor-performing component)
- Further evaluation remains to be done on the best algorithms to use in the consensus pipeline for particular dataset conditions

## Evidence

- [intro] CHAI-AvgSim is performed by aggregating all clustering assignments with an average similarity matrix, and performing Spectral Clustering: "CHAI-AvgSim is performed by aggregating all clustering assignments with an average similarity matrix, and performing Spectral Clustering"
- [methods] Binary similarity construction: "If two cells have the same clustering assignment, assign a value of 1 to a binary similarity matrix corresponding to the two cells."
- [methods] Silhouette-guided k selection: [evidence span withheld — anchor status: not_found]
- [methods] Aggregation step: "Each binary similarity matrix per algorithm is then aggregated into an Average Similarity matrix"
- [readme] eval flag semantics: "If "eval" is set to TRUE, CHAI will evaluate the best_k for Spectral Clustering using silhouette score."
