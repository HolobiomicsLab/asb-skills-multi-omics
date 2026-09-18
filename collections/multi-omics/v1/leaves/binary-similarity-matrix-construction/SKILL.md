---
name: binary-similarity-matrix-construction
description: Use when individual clustering assignments (e.g., from Seurat-Louvain,
  SC3, Spectrum, scSHC) must be converted into cell-to-cell binary similarity matrices
  before ensemble aggregation via averaging or SNF.
when_to_use_negative:
- When the goal is to evaluate a single clustering method directly against ground
  truth (ARI/NMI) — no similarity-matrix integration is needed
- When clustering assignments come from methods outside CHAI's seven and the user
  cannot produce cell-aligned cluster labels for every algorithm, since the binary
  matrices require identical cell orderings across all inputs
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: CHAI
  role: R package providing create_matrix_list to convert each algorithm's clustering
    assignment into a binary similarity matrix
  repo: https://github.com/lodimk2/chai
- name: Seurat
  role: Source of two constituent clusterings (Seurat-Louvain, Seurat-SLC) whose assignments
    become binary matrices
- name: SC3
  role: Constituent consensus clustering algorithm whose assignment is converted into
    a binary similarity matrix
- name: Spectrum
  role: Constituent clustering algorithm contributing one binary similarity matrix
- name: scSHC
  role: Statistical-significance-based constituent clustering algorithm contributing
    one binary similarity matrix
- name: SNFtool
  role: Provides the SNF function that fuses the binary similarity matrices into snf_matrix
    for CHAI-SNF
merged_aliases:
- binary-similarity-matrix-construction-from-cluster-assignments
- binary-similarity-matrix-integration
merged_alias_records:
- alias: binary-similarity-matrix-construction-from-cluster-assignments
  slug: binary-similarity-matrix-construction-from-cluster-assignments
  jaccard_score: 0.6667
  method: token-set-jaccard
  decision: needs_review
- alias: binary-similarity-matrix-integration
  slug: binary-similarity-matrix-integration
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_005
  - task_006
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:64606a65de2570add166323cf8077ec68275f53ec5a5f23a846bb25f58e41ca3
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9047619047619048
n_claims: 21
---

# binary-similarity-matrix-construction

## Summary

Constructs per-algorithm m x m binary similarity matrices from scRNAseq clustering assignments — entry 1 if two cells share a cluster, 0 otherwise — as the foundational aggregation input for the CHAI consensus framework (CHAI-AvgSim and CHAI-SNF).

## When to use

Use when individual clustering assignments (e.g., from Seurat-Louvain, SC3, Spectrum, scSHC) must be converted into cell-to-cell binary similarity matrices before ensemble aggregation via averaging or SNF.

## When NOT to use

- When the goal is to evaluate a single clustering method directly against ground truth (ARI/NMI) — no similarity-matrix integration is needed
- When clustering assignments come from methods outside CHAI's seven and the user cannot produce cell-aligned cluster labels for every algorithm, since the binary matrices require identical cell orderings across all inputs

## Inputs

- Per-algorithm clustering assignment table (Cell ID / cluster label) for each of the seven CHAI algorithms: Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC
- SingleCellExperiment object with counts and logcounts assays (e.g., baron_mouse_1 dataset)

## Outputs

- List of seven m x m binary similarity matrices (same cluster = 1, different cluster = 0), one per clustering algorithm (create_matrix_list output)
- Downstream aggregates: AvgSim_matrix (element-wise average) and/or snf_matrix (SNF-fused), consumed by Spectral Clustering

## How to apply

For each of the seven CHAI constituent algorithms (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC), take its Cell ID / Clustering Assignment table and build an m x m matrix over the m cells: assign 1 to the entry for two cells if they receive the same cluster label, and 0 otherwise. In the CHAI R package this is implemented by the create_matrix_list function, which accepts clustering assignments computed on a SingleCellExperiment object (counts plus logcounts, e.g., via scuttle::logNormCounts) and returns the list of binary matrices. These per-algorithm matrices are then aggregated either element-wise into the AvgSim_matrix (create_average_matrix) or fused via SNF (create_snf_matrix, SNFtool defaults) before Spectral Clustering. The rationale is a wisdom-of-crows design: each binary matrix encodes one algorithm's pairwise co-clustering vote, and integration of all seven votes yields the consensus. Verify each matrix is square m x m, symmetric, with entries in {0,1}, and that cell identifiers match the columns of the input SingleCellExperiment object.

## Related tools

- **CHAI** (R package providing create_matrix_list to convert each algorithm's clustering assignment into a binary similarity matrix) — https://github.com/lodimk2/chai
- **Seurat** (Source of two constituent clusterings (Seurat-Louvain, Seurat-SLC) whose assignments become binary matrices)
- **SC3** (Constituent consensus clustering algorithm whose assignment is converted into a binary similarity matrix)
- **Spectrum** (Constituent clustering algorithm contributing one binary similarity matrix)
- **scSHC** (Statistical-significance-based constituent clustering algorithm contributing one binary similarity matrix)
- **SNFtool** (Provides the SNF function that fuses the binary similarity matrices into snf_matrix for CHAI-SNF)

## Examples

```
devtools::install_github("lodimk2/chai"); data("baron_mouse_1"); sce <- SingleCellExperiment(assays=list(counts=as.matrix(baron_mouse_1))); sce <- scuttle::logNormCounts(sce); matrix_list <- chai::create_matrix_list(sce)
```

## Evaluation signals

- Each output matrix is square m x m (m = number of cells), symmetric, with all entries in {0,1}
- Diagonal entries are 1 (every cell shares its own cluster with itself) and cell identifiers/ordering match the input SingleCellExperiment columns exactly
- One binary matrix is produced per constituent algorithm (seven matrices for the default CHAI pipeline)
- Feeding the matrices into create_average_matrix yields an AvgSim_matrix with entries in [0,1] equal to the fraction of algorithms that co-cluster each cell pair

## Limitations

- CHAI currently has no method to select an ideal set of algorithms for the consensus pipeline; the article recommends including as many algorithms as possible
- Spectrum often displays subzero and negative ARI (a poor-performing component), which can inject noise into its binary similarity matrix
- The binary matrices only capture co-clustering agreement, not cluster-count confidence or probabilistic memberships

## Evidence

- [intro] Create binary similarity matrices from each clustering assignment (same cluster = 1, different cluster = 0): [evidence span withheld — anchor status: not_found]
- [methods] Binary matrix definition in methods: "If two cells have the same clustering assignment, assign a value of 1 to a binary similarity matrix corresponding to the two cells."
- [intro] Aggregation rationale: "CHAI-AvgSim is performed by aggregating all clustering assignments with an average similarity matrix, and performing Spectral Clustering"
- [readme] README data format for CHAI: [evidence span withheld — anchor status: external_doc]
- [intro] Seven constituent algorithms: "CHAI contains seven state-of-the-art scRNAseq clustering algorithms (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, and scSHC)"
