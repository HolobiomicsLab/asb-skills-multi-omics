---
name: spectral-clustering-consensus-assignment
description: Use when multiple scRNAseq clustering assignments (e.g., Seurat-Louvain,
  Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC) for the same cells must be fused
  into one consensus clustering, and the true number of clusters is unknown.
when_to_use_negative:
- When the goal is fusion via Similarity Network Fusion rather than average similarity
  — use the CHAI-SNF wrapper on the snf_matrix instead
- When the consensus clustering must include spatial transcriptomics data or multiomic
  integration that CHAI-SNF-ST handles better than AvgSim
- When inputs are raw per-cell expression matrices without any per-algorithm clustering
  assignments — the constituent algorithms must be run first to produce assignment
  tables
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CHAI
  role: R package implementing the AvgSim consensus pipeline (binary similarity matrices,
    averaging, silhouette-based best_k, Spectral Clustering)
  repo: https://github.com/lodimk2/chai
- name: CHAI-AvgSim
  role: Wrapper function that aggregates clustering assignments with an average similarity
    matrix and performs Spectral Clustering
  repo: https://github.com/lodimk2/chai
- name: R
  role: Language in which the CHAI package is written and executed
- name: scSHC
  role: One of the seven constituent scRNAseq clustering algorithms whose assignments
    feed the similarity matrix aggregation
- name: SC3
  role: One of the seven constituent scRNAseq clustering algorithms whose assignments
    feed the similarity matrix aggregation
- name: Spectrum
  role: One of the seven constituent scRNAseq clustering algorithms whose assignments
    feed the similarity matrix aggregation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:26d44661a1c6813e25383a5166432eeba4435157c03fea4fa07cdb740982c51a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lodimk2/chai
  tools:
  - CHAI
  - CHAI-AvgSim
  - R
  - SC3
  - Spectrum
  - scSHC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# spectral-clustering-consensus-assignment

## Summary

Applies the CHAI-AvgSim aggregation path: converts per-algorithm scRNAseq clustering assignments into binary similarity matrices, averages them into an AvgSim_matrix, selects best_k via silhouette scores, and runs Spectral Clustering to obtain a single consensus clustering of cells.

## When to use

Use when multiple scRNAseq clustering assignments (e.g., Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC) for the same cells must be fused into one consensus clustering, and the true number of clusters is unknown.

## When NOT to use

- When the goal is fusion via Similarity Network Fusion rather than average similarity — use the CHAI-SNF wrapper on the snf_matrix instead
- When the consensus clustering must include spatial transcriptomics data or multiomic integration that CHAI-SNF-ST handles better than AvgSim
- When inputs are raw per-cell expression matrices without any per-algorithm clustering assignments — the constituent algorithms must be run first to produce assignment tables

## Inputs

- SingleCellExperiment object with counts and logcounts assays (e.g., baron_mouse_1 from the Baron Mouse 1 dataset)
- Per-algorithm clustering assignment tables for the same cells (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC)
- Candidate best_k value (used as the upper anchor of the silhouette evaluation range 2 to k+1 when eval=TRUE)

## Outputs

- AvgSim_matrix: element-wise average of the per-algorithm binary similarity matrices
- best_k: cluster count with the maximum silhouette score over the evaluated range
- consensus_assignments.csv: one consensus cluster label per cell from Spectral Clustering on the AvgSim_matrix

## How to apply

Run each constituent clustering algorithm on the SingleCellExperiment object (counts plus logcounts, e.g. via scuttle::logNormCounts), then convert each assignment table into an m x m binary similarity matrix (create_matrix_list) where two cells sharing a cluster get 1 and otherwise 0. Element-wise average all binary matrices (create_average_matrix) to build the AvgSim_matrix. When the true cluster count is unknown, compute silhouette scores for candidate cluster counts in the range 2 to k+1 (calc_silhouette_scores) and set best_k to the value with the highest silhouette score; otherwise supply best_k directly to CHAI_AvgSim with eval=FALSE. Finally run Spectral Clustering on the AvgSim_matrix with best_k clusters to determine the final consensus cell identities.

## Related tools

- **CHAI** (R package implementing the AvgSim consensus pipeline (binary similarity matrices, averaging, silhouette-based best_k, Spectral Clustering)) — https://github.com/lodimk2/chai
- **CHAI-AvgSim** (Wrapper function that aggregates clustering assignments with an average similarity matrix and performs Spectral Clustering) — https://github.com/lodimk2/chai
- **R** (Language in which the CHAI package is written and executed)
- **scSHC** (One of the seven constituent scRNAseq clustering algorithms whose assignments feed the similarity matrix aggregation)
- **SC3** (One of the seven constituent scRNAseq clustering algorithms whose assignments feed the similarity matrix aggregation)
- **Spectrum** (One of the seven constituent scRNAseq clustering algorithms whose assignments feed the similarity matrix aggregation)

## Examples

```
best_k <- 15; sce <- CHAI_AvgSim(sce, best_k, eval = TRUE)
```

## Evaluation signals

- consensus_assignments.csv exists and contains exactly one consensus cluster label for every cell in the input dataset (e.g., baron_mouse_1)
- The recorded best_k attains the maximum silhouette score among all candidates evaluated in the range 2 to k+1
- Each per-algorithm binary similarity matrix is m x m with entries restricted to 0/1, and the AvgSim_matrix equals the element-wise average across all seven algorithms
- The AvgSim_matrix diagonal should equal 1 since every cell is in the same cluster as itself in each algorithm's assignment
- CHAI-AvgSim and CHAI-SNF should outperform SAME-clustering (ARI comparison) on benchmarking datasets when evaluated against ground truth

## Limitations

- It is not currently possible to select an ideal set of algorithms to be used in the final consensus; the authors' recommendation is to include as many algorithms as possible pending community best practices
- Individual clustering algorithms demonstrate large variation in performance, and Spectrum often displays subzero and negative ARI (a poor-performing component)
- Further evaluation remains to be done on the best algorithms to use in the consensus pipeline for particular dataset conditions

## Evidence

- [intro] CHAI-AvgSim is performed by aggregating all clustering assignments with an average similarity matrix, and performing Spectral Clustering: "CHAI-AvgSim is performed by aggregating all clustering assignments with an average similarity matrix, and performing Spectral Clustering"
- [methods] Binary similarity matrix construction rule: "If two cells have the same clustering assignment, assign a value of 1 to a binary similarity matrix corresponding to the two cells."
- [methods] Silhouette-based best_k selection range: [evidence span withheld — anchor status: not_found]
- [readme] README example invocation of CHAI_AvgSim with eval flag semantics: "If "eval" is set to TRUE, CHAI will evaluate the best_k for Spectral Clustering using silhouette score."
- [methods] Final spectral clustering step on the integrated matrix: "(iii) Run Spectral Clustering on either integrated matrix to determine final cell identities."
