---
name: scrnaseq-clustering-ensemble-implementation
description: Use when multiple scRNAseq clustering results must be integrated into
  one consensus cell-assignment set via average similarity on binary matrices, with
  silhouette-guided k selection.
when_to_use_negative:
- When a similarity-network-fusion aggregation is preferred over average similarity
  — CHAI-SNF applies the SNF algorithm across the binary similarity matrices instead
- When an EM mixture-model ensemble is needed, e.g. SAME-clustering, which builds
  its consensus from SC3, CIDR, Seurat, t-SNE + k-means and SIMLR solutions rather
  than averaged binary similarity matrices
- When the input is a raw count matrix and no individual clustering assignments have
  been generated yet — individual clustering must run first
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CHAI
  role: 'R package providing the AvgSim ensemble: create_matrix_list, create_average_matrix,
    calc_silhouette_scores, and CHAI_AvgSim wrapper'
  repo: https://github.com/lodimk2/chai
- name: R
  role: Implementation language; CHAI is an open source R package installed via devtools::install_github
  repo: https://github.com/lodimk2/chai
- name: Seurat
  role: Contributes Seurat-Louvain and Seurat-SLC individual clustering assignments
    to the ensemble
- name: SC3
  role: One of the seven component clustering methods whose binary similarity matrix
    is averaged
- name: scSHC
  role: One of the seven component clustering methods (statistical significance testing
    based) feeding the AvgSim aggregation
- name: Spectrum
  role: One of the seven component clustering methods aggregated into the consensus
- name: CHOIR
  role: One of the seven component clustering methods aggregated into the consensus
- name: RACEID
  role: One of the seven component clustering methods aggregated into the consensus
- name: SAME-clustering
  role: Baseline consensus clustering method used for benchmarking comparison
  repo: https://github.com/yycunc/SAMEclustering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:64ebce2ef15eee77881040c5593beb4bf1770a016940168dfb17805985b5d47b
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
  - SAME-clustering
  - SC3
  - Seurat
  - Spectrum
  - scSHC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# scrnaseq-clustering-ensemble-implementation

## Summary

Implements CHAI-AvgSim, a wisdom-of-crowds consensus clustering approach that converts seven scRNAseq clustering assignments (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC) into binary similarity matrices, averages them element-wise, and runs Spectral Clustering to produce a single consensus clustering of cells.

## When to use

Use when multiple scRNAseq clustering results must be integrated into one consensus cell-assignment set via average similarity on binary matrices, with silhouette-guided k selection.

## When NOT to use

- When a similarity-network-fusion aggregation is preferred over average similarity — CHAI-SNF applies the SNF algorithm across the binary similarity matrices instead
- When an EM mixture-model ensemble is needed, e.g. SAME-clustering, which builds its consensus from SC3, CIDR, Seurat, t-SNE + k-means and SIMLR solutions rather than averaged binary similarity matrices
- When the input is a raw count matrix and no individual clustering assignments have been generated yet — individual clustering must run first

## Inputs

- SingleCellExperiment object with counts and logcounts assays (e.g. baron_mouse_1 bundled dataset)
- Clustering assignment tables from seven methods: Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC
- Candidate k value for Spectral Clustering (best_k parameter, README example best_k <- 15)

## Outputs

- consensus_assignments.csv with one consensus cluster label per cell
- AvgSim_matrix (m x m element-wise averaged similarity matrix)
- Recorded best_k (cluster count with highest silhouette score in range 2 to k+1)

## How to apply

Run each of the seven bundled clustering algorithms on the SingleCellExperiment object (counts plus logcounts, e.g. from scuttle::logNormCounts), then convert each clustering assignment table into an m x m binary similarity matrix (create_matrix_list), assigning 1 when two cells share a cluster and 0 otherwise. Aggregate the binary matrices element-wise via create_average_matrix to obtain the AvgSim_matrix, the per-element average across all algorithms. If the true cluster number is unknown, set eval=TRUE so CHAI computes silhouette scores for candidate cluster counts in the range 2 to k+1 (calc_silhouette_scores) and picks best_k with the maximum silhouette. Run Spectral Clustering on the AvgSim_matrix with best_k clusters to determine final consensus cell identities. Rationale: averaging binary similarity across diverse methods exploits the wisdom of crowds, since no single scRNAseq method is best across all scenarios.

## Related tools

- **CHAI** (R package providing the AvgSim ensemble: create_matrix_list, create_average_matrix, calc_silhouette_scores, and CHAI_AvgSim wrapper) — https://github.com/lodimk2/chai
- **R** (Implementation language; CHAI is an open source R package installed via devtools::install_github) — https://github.com/lodimk2/chai
- **Seurat** (Contributes Seurat-Louvain and Seurat-SLC individual clustering assignments to the ensemble)
- **SC3** (One of the seven component clustering methods whose binary similarity matrix is averaged)
- **scSHC** (One of the seven component clustering methods (statistical significance testing based) feeding the AvgSim aggregation)
- **Spectrum** (One of the seven component clustering methods aggregated into the consensus)
- **CHOIR** (One of the seven component clustering methods aggregated into the consensus)
- **RACEID** (One of the seven component clustering methods aggregated into the consensus)
- **SAME-clustering** (Baseline consensus clustering method used for benchmarking comparison) — https://github.com/yycunc/SAMEclustering

## Examples

```
sce <- SingleCellExperiment(assays = list(counts = as.matrix(baron_mouse_1))); sce <- scuttle::logNormCounts(sce); best_k <- 15; sce <- CHAI_AvgSim(sce, best_k, eval = TRUE)
```

## Evaluation signals

- consensus_assignments.csv exists and contains exactly one consensus cluster label for every cell in the input SingleCellExperiment (baron_mouse_1)
- The AvgSim_matrix equals the element-wise average of the seven per-method binary similarity matrices
- Recorded best_k attains the maximum silhouette score among candidate cluster counts evaluated in the range 2 to k+1
- Downstream ARI on benchmark datasets (e.g. Baron Mouse 1, Zheng68K PBMC subsamples) shows CHAI-AvgSim outperforming SAME-clustering
- Each binary similarity matrix is symmetric with 1 only where two cells share the same cluster and 0 otherwise

## Limitations

- No method currently exists to select an ideal set of algorithms for the CHAI consensus pipeline; current recommendation is to include as many algorithms as possible
- Individual clustering algorithms demonstrate large variation in performance; Spectrum often displays subzero and negative ARI (a poor-performing component)
- SC3 is included in CHAI's pipeline but not in SAME-Clustering's in all evaluations, complicating benchmarking comparisons
- SC3 does not run in the SAME-clustering package due to a bug (github.com/yycunc/SAMEclustering/issues/4)

## Evidence

- [intro] CHAI-AvgSim is performed by aggregating all clustering assignments with an average similarity matrix, and performing Spectral Clustering: "CHAI-AvgSim is performed by aggregating all clustering assignments with an average similarity matrix, and performing Spectral Clustering"
- [intro] If two cells are assigned the same cluster, assign a score of 1.: "If two cells are assigned the same cluster, assign a score of 1."
- [methods] silhouette evaluation range: [evidence span withheld — anchor status: not_found]
- [intro] CHAI contains two clustering ensemble approaches, named CHAI-AvgSim and CHAI-SNF: "CHAI contains two clustering ensemble approaches, named CHAI-AvgSim and CHAI-SNF"
- [readme] eval flag for best_k: "If "eval" is set to TRUE, CHAI will evaluate the best_k for Spectral Clustering using silhouette score."
