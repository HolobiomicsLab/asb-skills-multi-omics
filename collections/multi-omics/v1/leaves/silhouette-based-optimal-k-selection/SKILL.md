---
name: silhouette-based-optimal-k-selection
description: Use when running CHAI-AvgSim or CHAI-SNF on a SingleCellExperiment (e.g.,
  baron_mouse_1) and the true number of clusters is unknown, so a data-driven highest-ranked_k
  must be selected before Spectral Clustering on the AvgSim_matrix or snf_matrix.
when_to_use_negative:
- The true number of clusters is already known and the user wants to fix k directly
  — set eval = FALSE and supply best_k rather than running silhouette evaluation
- The input is raw count data without the upstream CHAI binary similarity matrices
  and average matrix (AvgSim_matrix) computed — silhouette evaluation of consensus
  assignments presupposes the CHAI-AvgSim/CHAI-SNF aggregation path
- Selecting the number of clusters for a single individual clustering algorithm (e.g.,
  SC3 or Seurat) rather than for the CHAI consensus Spectral Clustering step
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CHAI
  role: R package whose CHAI_AvgSim/CHAI_SNF wrappers perform silhouette-based best_k
    evaluation and subsequent Spectral Clustering
  repo: https://github.com/lodimk2/chai
- name: CHAI-AvgSim
  role: Ensemble approach that averages binary similarity matrices, evaluates silhouette
    scores over candidate k, and runs Spectral Clustering with the selected best_k
  repo: https://github.com/lodimk2/chai
- name: CHAI-SNF
  role: Alternative ensemble approach using the snf_matrix; supports the same eval
    = TRUE silhouette-based best_k selection
  repo: https://github.com/lodimk2/chai
- name: R
  role: Implementation language of the CHAI package and its silhouette evaluation
    functions
  repo: https://github.com/lodimk2/chai
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:7d9dd6abc777b7773e7e6b52c4915de8847950bf063c0a0f608bda6ed2fba32f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lodimk2/chai
  tools:
  - CHAI
  - CHAI-AvgSim
  - CHAI-SNF
  - R
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# silhouette-based-optimal-k-selection

## Summary

Determines the number of clusters (best_k) for Spectral Clustering in the CHAI consensus pipeline by evaluating silhouette scores across candidate cluster counts, used when the true number of cell types in an scRNAseq dataset is unknown.

## When to use

Use when running CHAI-AvgSim or CHAI-SNF on a SingleCellExperiment (e.g., baron_mouse_1) and the true number of clusters is unknown, so a data-driven best_k must be selected before Spectral Clustering on the AvgSim_matrix or snf_matrix.

## When NOT to use

- The true number of clusters is already known and the user wants to fix k directly — set eval = FALSE and supply best_k rather than running silhouette evaluation
- The input is raw count data without the upstream CHAI binary similarity matrices and average matrix (AvgSim_matrix) computed — silhouette evaluation of consensus assignments presupposes the CHAI-AvgSim/CHAI-SNF aggregation path
- Selecting the number of clusters for a single individual clustering algorithm (e.g., SC3 or Seurat) rather than for the CHAI consensus Spectral Clustering step

## Inputs

- SingleCellExperiment object with counts and logcounts assays (e.g., baron_mouse_1 from the Baron datasets)
- Aggregated AvgSim_matrix (element-wise average of the seven binary similarity matrices)
- User-supplied initial best_k value (e.g., 15) as the upper anchor of the candidate range 2 to k+1

## Outputs

- best_k: integer number of clusters with the maximum silhouette score in the evaluated range
- Consensus cluster assignments for every cell, produced by Spectral Clustering on the AvgSim_matrix with best_k clusters (consensus_assignments.csv)

## How to apply

Run the seven individual scRNAseq clustering algorithms (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC), convert each assignment table into an m x m binary similarity matrix, and aggregate them into the AvgSim_matrix via create_average_matrix. Then compute silhouette scores for candidate cluster counts over the range 2 to k+1, with k being the true number of clusters present in the dataset (calc_silhouette_scores), and set best_k to the value attaining the highest silhouette score. In the CHAI R wrapper this is controlled by the eval argument: setting eval = TRUE triggers automatic best_k evaluation by silhouette score, while eval = FALSE uses the user-supplied best_k directly. Finally, run Spectral Clustering on the AvgSim_matrix (or snf_matrix for CHAI-SNF) with best_k clusters to obtain consensus cell identities. Rationale: the silhouette statistic measures how well each cell fits its assigned consensus cluster versus neighbors, so maximizing it over the candidate range selects a cluster count consistent with the aggregated similarity structure.

## Related tools

- **CHAI** (R package whose CHAI_AvgSim/CHAI_SNF wrappers perform silhouette-based best_k evaluation and subsequent Spectral Clustering) — https://github.com/lodimk2/chai
- **CHAI-AvgSim** (Ensemble approach that averages binary similarity matrices, evaluates silhouette scores over candidate k, and runs Spectral Clustering with the selected best_k) — https://github.com/lodimk2/chai
- **CHAI-SNF** (Alternative ensemble approach using the snf_matrix; supports the same eval = TRUE silhouette-based best_k selection) — https://github.com/lodimk2/chai
- **R** (Implementation language of the CHAI package and its silhouette evaluation functions) — https://github.com/lodimk2/chai

## Examples

```
best_k <- 15; sce <- CHAI_AvgSim(sce, best_k, eval = TRUE)
```

## Evaluation signals

- consensus_assignments.csv exists and contains exactly one consensus cluster label for every cell in baron_mouse_1
- The recorded best_k attains the maximum silhouette score among all candidate counts in the evaluated range 2 to k+1
- With eval = TRUE, the wrapper output matches the best_k reported by calc_silhouette_scores rather than the user-supplied default
- Number of distinct consensus labels equals best_k after Spectral Clustering on the AvgSim_matrix
- All seven individual clustering assignments were converted to binary similarity matrices (1 for same cluster, 0 otherwise) before averaging

## Limitations

- The silhouette evaluation range depends on the user-supplied k parameter (search is 2 to k+1), so an ill-chosen anchor can exclude the true cluster count
- CHAI cannot currently select an ideal subset of the seven component algorithms, and individual algorithms such as Spectrum often display subzero and negative ARI, which can degrade the similarity matrix that silhouette scores are computed on
- Further evaluation remains to be done on the best algorithms to use in the consensus pipeline for particular dataset conditions

## Evidence

- [intro] users can determine the "best_k" for Spectral Clustering by evaluating the optimal silhouette score: "users can determine the "best_k" for Spectral Clustering by evaluating the optimal silhouette score"
- [methods] silhouette range 2 to k+1: [evidence span withheld — anchor status: not_found]
- [readme] eval argument controls evaluation: "If "eval" is set to TRUE, CHAI will evaluate the best_k for Spectral Clustering using silhouette score. If "eval" is set to false, CHAI will use best_k as the k value for silhouette score."
- [intro] Spectral Clustering on averaged matrix: "CHAI-AvgSim is performed by aggregating all clustering assignments with an average similarity matrix, and performing Spectral Clustering"
