---
name: cdh3-tumor-cluster-marker-validation
description: Use when a scRNA-seq dataset requires a consensus clustering consensus
  followed by marker-based identification of a biologically meaningful cell population,
  e.g. detecting a tumor leader cell cluster enriched for a specific marker such as
  CDH3.
when_to_use_negative:
- The goal is to compare consensus clustering methods by ARI on benchmarking data
  with ground-truth labels (e.g. sc_mixology cell line mixtures) rather than to identify
  a biologically meaningful cluster by marker enrichment
- The input dataset lacks a defined candidate marker gene, so cluster validation by
  marker enrichment (e.g. CDH3 upregulation in exactly one cluster) cannot serve as
  the acceptance criterion
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CHAI
  role: R package that runs the seven component clustering algorithms and provides
    the CHAI-AvgSim wrapper for consensus clustering
  repo: https://github.com/lodimk2/chai
- name: CHAI-AvgSim
  role: Aggregates binary similarity matrices from the seven clusterers via average
    similarity and performs Spectral Clustering with silhouette-score-based best_k
    selection
  repo: https://github.com/lodimk2/chai
- name: Seurat
  role: Provides the FindAllMarkers function used to compute per-cluster differentially
    expressed genes on the CHAI consensus clusters
- name: SC3
  role: One of the seven component clustering algorithms contributing a binary similarity
    matrix to the consensus
- name: Spectrum
  role: One of the seven component clustering algorithms contributing a binary similarity
    matrix to the consensus
- name: scSHC
  role: One of the seven component clustering algorithms contributing a binary similarity
    matrix to the consensus
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:0d92b6def6c0d45a2bd2d8e74f58ab2fc0bcd77d345c4186277664acb0ad9a72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lodimk2/chai
  tools:
  - CHAI
  - CHAI-AvgSim
  - R
  - SC3
  - Seurat
  - Spectrum
  - scSHC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# cdh3-tumor-cluster-marker-validation

## Summary

A reproducible use case for the CHAI consensus clustering framework that identifies a CDH3-enriched leader tumor cell population in the Savas breast cancer (TNBC) CD3+ scRNA-seq dataset (SCP2331, 5759 cells) by running the seven CHAI component clusterers, aggregating binary similarity matrices via CHAI-AvgSim, and validating the tumor cluster through Seurat FindAllMarkers.

## When to use

Use when a scRNA-seq dataset requires a consensus clustering consensus followed by marker-based identification of a biologically meaningful cell population, e.g. detecting a tumor leader cell cluster enriched for a specific marker such as CDH3.

## When NOT to use

- The goal is to compare consensus clustering methods by ARI on benchmarking data with ground-truth labels (e.g. sc_mixology cell line mixtures) rather than to identify a biologically meaningful cluster by marker enrichment
- The input dataset lacks a defined candidate marker gene, so cluster validation by marker enrichment (e.g. CDH3 upregulation in exactly one cluster) cannot serve as the acceptance criterion

## Inputs

- Savas Breast Cancer (TNBC) CD3+ single-cell RNA-seq dataset from Broad Single Cell Portal accession SCP2331 (5759 cells)
- SingleCellExperiment object with counts and logcounts assays defined

## Outputs

- cluster_assignments.csv with one CHAI consensus cluster assignment per cell for all 5759 cells
- marker_summary.csv with per-cluster differentially expressed genes from Seurat FindAllMarkers
- The cluster in which CDH3 is the top enriched marker, reported as the tumor leader cell cluster

## How to apply

Load the Savas Breast Cancer (TNBC) CD3+ dataset (Broad Single Cell Portal accession SCP2331, 5759 cells) into R as a SingleCellExperiment object with counts and logcounts defined. Run the seven CHAI component clustering algorithms (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC) and convert each clustering assignment into a binary similarity matrix (same cluster = 1, different cluster = 0). Aggregate the binary matrices into the Average Similarity matrix (CHAI-AvgSim) and select best_k for Spectral Clustering by evaluating the optimal silhouette score, then run Spectral Clustering on the AvgSim_matrix to obtain final consensus cell identities. Compute per-cluster differentially expressed genes with Seurat's FindAllMarkers and identify the single cluster in which CDH3 is the top significantly enriched marker, reporting that cluster as the tumor leader cell cluster. The recommendation from the authors is to include as many component algorithms as possible in the consensus pipeline.

## Related tools

- **CHAI** (R package that runs the seven component clustering algorithms and provides the CHAI-AvgSim wrapper for consensus clustering) — https://github.com/lodimk2/chai
- **CHAI-AvgSim** (Aggregates binary similarity matrices from the seven clusterers via average similarity and performs Spectral Clustering with silhouette-score-based best_k selection) — https://github.com/lodimk2/chai
- **Seurat** (Provides the FindAllMarkers function used to compute per-cluster differentially expressed genes on the CHAI consensus clusters)
- **SC3** (One of the seven component clustering algorithms contributing a binary similarity matrix to the consensus)
- **Spectrum** (One of the seven component clustering algorithms contributing a binary similarity matrix to the consensus)
- **scSHC** (One of the seven component clustering algorithms contributing a binary similarity matrix to the consensus)

## Examples

```
sce <- SingleCellExperiment(assays = list(counts = as.matrix(savas_scp2331))); sce <- scuttle::logNormCounts(sce); sce <- CHAI_AvgSim(sce, best_k = 15, eval = TRUE)
```

## Evaluation signals

- marker_summary.csv shows CDH3 significantly upregulated in exactly one CHAI cluster relative to all other clusters
- cluster_assignments.csv contains exactly one assignment per cell for all 5759 cells from the Savas SCP2331 dataset
- best_k for Spectral Clustering was selected by evaluating the optimal silhouette score (eval = TRUE)
- The AvgSim_matrix was correctly built from seven binary similarity matrices (one per component algorithm: Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC)
- The SingleCellExperiment object contains both counts and logcounts assays before running the CHAI wrappers

## Limitations

- It is not currently possible to select an ideal set of algorithms to be used in the final consensus; the authors recommend including as many algorithms as possible
- Individual clustering algorithms demonstrate large variation in performance, and Spectrum often displays subzero and negative ARI (poor-performing component)
- Further evaluation remains to be done on the best algorithms to use in the consensus pipeline for particular dataset conditions

## Evidence

- [abstract] We demonstrate CHAI's practical use case by identifying a leader tumor cell cluster enriched with CDH3: "We demonstrate CHAI's practical use case by identifying a leader tumor cell cluster enriched with CDH3"
- [intro] AvgSim aggregation and spectral clustering: "CHAI-AvgSim is performed by aggregating all clustering assignments with an average similarity matrix, and performing Spectral Clustering"
- [intro] best_k selection by silhouette: "users can determine the "best_k" for Spectral Clustering by evaluating the optimal silhouette score"
- [readme] README data format requirement: [evidence span withheld — anchor status: external_doc]
- [readme] README eval flag: "If "eval" is set to TRUE, CHAI will evaluate the best_k for Spectral Clustering using silhouette score."
- [methods] Binary similarity matrix construction: "If two cells have the same clustering assignment, assign a value of 1 to a binary similarity matrix corresponding to the two cells."
