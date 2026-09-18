---
name: silhouette-score-k-selection
description: Use when running CHAI-AvgSim or CHAI-SNF on scRNAseq data where the true
  number of clusters is unknown and a highest-ranked_k must be selected for Spectral
  Clustering via silhouette score.
when_to_use_negative:
- The true number of clusters is already known and the user wishes to fix best_k directly
  (set eval=FALSE in CHAI instead of re-evaluating silhouette scores)
- Clustering is being done by an individual component algorithm (e.g., SC3, Spectrum)
  rather than on the integrated AvgSim or SNF matrix — silhouette evaluation in the
  2 to k+1 range is defined for CHAI's integrated matrices
- Input is spatial transcriptomics processed via CHAI-SNF-ST, where the aggregation
  path includes additional spatial data and best_k selection may need separate consideration
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CHAI
  role: R package providing CHAI_AvgSim and CHAI_SNF wrappers with an eval flag that
    triggers silhouette-score evaluation of best_k for Spectral Clustering
  repo: https://github.com/lodimk2/chai
- name: SNFtool
  role: Produces the snf_matrix whose distinct similarity distribution requires a
    separate silhouette-based best_k evaluation from the AvgSim matrix
- name: CHAI-SNF
  role: Aggregation path that fuses binary similarity matrices via SNF; best_k for
    spectral clustering on the snf_matrix is evaluated separately by silhouette score
  repo: https://github.com/lodimk2/chai
- name: CHAI-AvgSim
  role: Aggregation path producing the AvgSim_matrix on which silhouette-based best_k
    selection is performed before Spectral Clustering
  repo: https://github.com/lodimk2/chai
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:5df5efc1f1f6f89dc47d763b169ec165ea548f2688bebd4f10fa67883564513d
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
  edam_operation: http://edamontology.org/operation_3644
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# silhouette-score-k-selection

## Summary

Determines the optimal number of clusters (best_k) for Spectral Clustering in the CHAI consensus clustering framework by evaluating average silhouette scores over candidate k values (range 2 to k+1), applied separately to the AvgSim_matrix and the SNF-fused snf_matrix.

## When to use

Use when running CHAI-AvgSim or CHAI-SNF on scRNAseq data where the true number of clusters is unknown and a best_k must be selected for Spectral Clustering via silhouette score.

## When NOT to use

- The true number of clusters is already known and the user wishes to fix best_k directly (set eval=FALSE in CHAI instead of re-evaluating silhouette scores)
- Clustering is being done by an individual component algorithm (e.g., SC3, Spectrum) rather than on the integrated AvgSim or SNF matrix — silhouette evaluation in the 2 to k+1 range is defined for CHAI's integrated matrices
- Input is spatial transcriptomics processed via CHAI-SNF-ST, where the aggregation path includes additional spatial data and best_k selection may need separate consideration

## Inputs

- AvgSim_matrix (m x m average similarity matrix built from seven binary similarity matrices) for CHAI-AvgSim
- snf_matrix (m x m fused cell-to-cell similarity matrix produced by SNFtool over the seven binary similarity matrices) for CHAI-SNF
- Candidate k range: 2 to k+1 (k = true number of clusters when known)
- SingleCellExperiment object with counts and logcounts assays

## Outputs

- best_k (integer, the candidate k maximizing average silhouette score)
- Final consensus per-cell cluster assignments from Spectral Clustering run at best_k (e.g., cluster_assignments.csv / consensus_assignments.csv)

## How to apply

After aggregating the seven per-algorithm binary similarity matrices (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC) into either the Average Similarity matrix (CHAI-AvgSim) or the SNF-fused snf_matrix (CHAI-SNF via SNFtool), compute the average silhouette score for each candidate k in the range 2 to k+1, where k is the true number of clusters when known (as in benchmarking). Select best_k as the k with the maximum average silhouette score, then run Spectral Clustering on the integrated matrix using that best_k. Note that best_k is evaluated separately for the SNF matrix because its similarity distribution differs from the AvgSim matrix. In the CHAI R package, setting eval=TRUE triggers this silhouette-based evaluation; eval=FALSE uses the user-supplied best_k directly.

## Related tools

- **CHAI** (R package providing CHAI_AvgSim and CHAI_SNF wrappers with an eval flag that triggers silhouette-score evaluation of best_k for Spectral Clustering) — https://github.com/lodimk2/chai
- **SNFtool** (Produces the snf_matrix whose distinct similarity distribution requires a separate silhouette-based best_k evaluation from the AvgSim matrix)
- **CHAI-SNF** (Aggregation path that fuses binary similarity matrices via SNF; best_k for spectral clustering on the snf_matrix is evaluated separately by silhouette score) — https://github.com/lodimk2/chai
- **CHAI-AvgSim** (Aggregation path producing the AvgSim_matrix on which silhouette-based best_k selection is performed before Spectral Clustering) — https://github.com/lodimk2/chai

## Examples

```
sce <- CHAI_AvgSim(sce, best_k = 15, eval = TRUE)  # eval=TRUE triggers silhouette-based best_k selection
```

## Evaluation signals

- The chosen best_k corresponds to the maximum average silhouette score over the evaluated range 2 to k+1
- best_k is evaluated independently for the SNF matrix and the AvgSim matrix (their similarity distributions differ, so a single shared best_k is not acceptable)
- Every cell (e.g., all 5759 cells in the Savas SCP2331 dataset) receives exactly one consensus cluster label after Spectral Clustering at best_k
- In CHAI runs, silhouette evaluation only occurs when eval=TRUE; with eval=FALSE the supplied best_k is used without re-selection

## Limitations

- The silhouette range 2 to k+1 assumes k (the true number of clusters) is known, which holds in benchmarking but not in fully unsupervised practical applications like the Savas CDH3 leader-cell identification
- An immediate limitation of CHAI is that it is not currently possible to select an ideal set of algorithms for the consensus, and individual algorithms (e.g., Spectrum with subzero/negative ARI) show large performance variation, which propagates into the matrix whose silhouette is being evaluated

## Evidence

- [methods] silhouette score evaluation in range 2 to k + 1: [evidence span withheld — anchor status: not_found]
- [intro] best_k via silhouette when true k unknown: "users can determine the "best_k" for Spectral Clustering by evaluating the optimal silhouette score"
- [results] SNF best_k evaluated separately: [evidence span withheld — anchor status: not_found]
- [readme] README eval flag: "If "eval" is set to TRUE, CHAI will evaluate the best_k for Spectral Clustering using silhouette score."
- [methods] validation of best_k selection: [evidence span withheld — anchor status: not_found]
