---
name: single-cell-rna-seq-consensus-clustering
description: Use when a scRNAseq (or spatial transcriptomics) dataset in SingleCellExperiment
  form needs a robust cell-type clustering that outperforms any single algorithm,
  by ensembling seven base clusterers into one consensus via binary similarity matrices.
when_to_use_negative:
- When you only need a single algorithm's clustering — CHAI's value comes from ensembling
  seven algorithms, and individual algorithms show large performance variation within
  the ensemble
- When input data is not a SingleCellExperiment object with counts and logcounts defined
  (e.g. FPKM/CPM-only matrices as rejected by related ensemble tools)
- When the goal is a probabilistic mixture-model ensemble like SAME-clustering rather
  than similarity-matrix integration
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CHAI
  role: Consensus clustering R package providing CHAI_AvgSim and CHAI_SNF wrapper
    functions that build binary similarity matrices, aggregate them, and run Spectral
    Clustering
  repo: https://github.com/lodimk2/chai
- name: CHAI-SNF
  role: CHAI variant that fuses the seven binary similarity matrices with the SNF
    algorithm (create_snf_matrix) before Spectral Clustering
  repo: https://github.com/lodimk2/chai
- name: SNFtool
  role: R package supplying the SNF algorithm used to iteratively fuse the binary
    similarity matrices via message passing
- name: Seurat
  role: Provides two of the seven base clusterers (Seurat-Louvain and Seurat-SLC via
    local moving / smart local moving modularity optimization)
- name: SC3
  role: Base clusterer in the pipeline; uses a gene filter removing genes in less
    than ~6% of cells and CSPA-based consensus
- name: Spectrum
  role: Base clusterer in the CHAI pipeline
- name: SAME-clustering
  role: Benchmark ensemble method (EM-based mixture model over SC3, CIDR, Seurat,
    t-SNE+k-means, SIMLR) against which CHAI methods are compared
  repo: https://github.com/yycunc/SAMEclustering
- name: R
  role: Implementation language; CHAI is an open source R package installed via devtools::install_github
  repo: https://github.com/lodimk2/chai
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:300c9c6aacd6a3e3a758d7d62bc06f2064e9c376ee4f42fbaedda656f3259b9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lodimk2/chai
  tools:
  - CHAI
  - CHAI-SNF
  - R
  - SAME-clustering
  - SC3
  - SNFtool
  - Seurat
  - Spectrum
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# single-cell-rna-seq-consensus-clustering

## Summary

A consensus clustering approach (CHAI) that runs seven state-of-the-art scRNAseq clustering algorithms (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC), converts each clustering assignment into a binary cell-to-cell similarity matrix, aggregates them via average similarity (CHAI-AvgSim) or Similarity Network Fusion (CHAI-SNF), and applies Spectral Clustering to obtain final cell-type assignments.

## When to use

Use when a scRNAseq (or spatial transcriptomics) dataset in SingleCellExperiment form needs a robust cell-type clustering that outperforms any single algorithm, by ensembling seven base clusterers into one consensus via binary similarity matrices.

## When NOT to use

- When you only need a single algorithm's clustering — CHAI's value comes from ensembling seven algorithms, and individual algorithms show large performance variation within the ensemble
- When input data is not a SingleCellExperiment object with counts and logcounts defined (e.g. FPKM/CPM-only matrices as rejected by related ensemble tools)
- When the goal is a probabilistic mixture-model ensemble like SAME-clustering rather than similarity-matrix integration

## Inputs

- SingleCellExperiment object with counts and logcounts assays (e.g. baron_mouse_1 from the Baron Mouse 1 Dataset)
- Clustering assignments from seven algorithms: Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC
- Seven m x m binary similarity matrices (one per algorithm, entries 0/1)
- best_k value for Spectral Clustering (used as upper bound if eval=TRUE)

## Outputs

- snf_matrix (fused m x m cell-to-cell similarity matrix, entries in [0,1], written to snf_matrix.csv)
- Per-cell consensus cluster assignments (written to consensus_assignments.csv)
- best_k selected by maximum average silhouette score over the evaluated range

## How to apply

Run the seven individual clustering algorithms on the SingleCellExperiment object (counts and logcounts defined), then for each algorithm build an m x m binary similarity matrix where an entry is 1 if two cells share a cluster assignment and 0 otherwise. For CHAI-SNF, apply the SNF algorithm (SNFtool R package, create_snf_matrix) across all seven binary similarity matrices, iteratively fusing the cell-to-cell similarity networks via message passing until convergence to yield snf_matrix; for CHAI-AvgSim, simply average the matrices. If the true number of clusters is unknown (eval=TRUE), determine best_k for Spectral Clustering by evaluating the optimal average silhouette score over the range 2 to k+1, where k is the true cluster number in benchmarking; note best_k must be re-evaluated separately for the SNF matrix because its distribution differs from the AvgSim matrix. Finally run Spectral Clustering on the integrated matrix (snf_matrix or AvgSim_matrix) with best_k to produce per-cell consensus cluster labels.

## Related tools

- **CHAI** (Consensus clustering R package providing CHAI_AvgSim and CHAI_SNF wrapper functions that build binary similarity matrices, aggregate them, and run Spectral Clustering) — https://github.com/lodimk2/chai
- **CHAI-SNF** (CHAI variant that fuses the seven binary similarity matrices with the SNF algorithm (create_snf_matrix) before Spectral Clustering) — https://github.com/lodimk2/chai
- **SNFtool** (R package supplying the SNF algorithm used to iteratively fuse the binary similarity matrices via message passing)
- **Seurat** (Provides two of the seven base clusterers (Seurat-Louvain and Seurat-SLC via local moving / smart local moving modularity optimization))
- **SC3** (Base clusterer in the pipeline; uses a gene filter removing genes in less than ~6% of cells and CSPA-based consensus)
- **Spectrum** (Base clusterer in the CHAI pipeline)
- **SAME-clustering** (Benchmark ensemble method (EM-based mixture model over SC3, CIDR, Seurat, t-SNE+k-means, SIMLR) against which CHAI methods are compared) — https://github.com/yycunc/SAMEclustering
- **R** (Implementation language; CHAI is an open source R package installed via devtools::install_github) — https://github.com/lodimk2/chai

## Examples

```
sce <- CHAI_SNF(sce, best_k = 15, eval = TRUE)
```

## Evaluation signals

- snf_matrix is an m x m cell-to-cell similarity matrix with all entries in [0,1]
- Every cell receives exactly one consensus cluster label in consensus_assignments.csv
- Chosen best_k corresponds to the maximum average silhouette score over candidate k values in range 2 to k+1
- best_k was evaluated separately on the SNF matrix (its distribution differs from the AvgSim matrix), not reused from the AvgSim evaluation
- Consensus ARI outperforms individual base clusterers and SAME-clustering on benchmarking datasets (e.g. sc_mixology, Zheng68K, Baron)

## Limitations

- No method exists to select an ideal set of algorithms for the CHAI consensus pipeline; current recommendation is to include as many algorithms as possible
- Individual clustering algorithms demonstrate large variation in performance; Spectrum often displays subzero and negative ARI (poor-performing component)
- Further evaluation remains to be done on the best algorithms to use in the consensus pipeline for particular dataset conditions
- SC3 does not run within the current SAME-Clustering package due to a bug, complicating fair comparison (SC3 is included in CHAI's pipeline but not in SAME-Clustering's in all evaluations)

## Evidence

- [methods] we apply the SNF algorithm across all binary similarity matrices [21]: "we apply the SNF algorithm across all binary similarity matrices [21]"
- [methods] binary similarity matrix construction: "If two cells have the same clustering assignment, assign a value of 1 to a binary similarity matrix corresponding to the two cells."
- [intro] best_k determination via silhouette: "users can determine the "best_k" for Spectral Clustering by evaluating the optimal silhouette score"
- [methods] silhouette evaluation range: [evidence span withheld — anchor status: not_found]
- [readme] two CHAI consensus methods: [evidence span withheld — anchor status: external_doc]
- [abstract] seven ensemble members: "CHAI presents two competing methods which aggregate the clustering results from seven state-of-the-art clustering methods: CHAI-AvgSim and CHAI-SNF"
