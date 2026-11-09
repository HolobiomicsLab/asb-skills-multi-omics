---
name: r-package-scripting
description: Use when scripting an R-based consensus clustering pipeline for scRNAseq
  data packaged as a SingleCellExperiment object, or when extending CHAI with custom
  clustering algorithms via its R API.
when_to_use_negative:
- Input data are not in R SingleCellExperiment format (e.g. raw FASTQ, mzML, or a
  Python AnnData object) — the CHAI wrappers require counts and logcounts assays
- The goal is to run a single clustering method rather than aggregate results from
  seven algorithms into a consensus
- Data are other formats such as FPKM, CPM and RPKM, which the analogous SAMEclustering
  package no longer accepts — use raw count data
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: CHAI
  role: The R package being scripted; provides CHAI_AvgSim and CHAI_SNF wrapper functions
    for consensus clustering
  repo: https://github.com/lodimk2/chai
- name: R
  role: Language in which the CHAI package is written and installed via devtools::install_github
- name: SNFtool
  role: Supplies the SNF algorithm applied across the seven binary similarity matrices
    in CHAI-SNF
- name: SAME-clustering
  role: Alternative R cluster-ensemble package used as the comparison baseline; has
    a bug preventing SC3 from running
  repo: https://github.com/yycunc/SAMEclustering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:f022992ef7235bf77c2057da49bd131bae3bd83764c5be371bcdf53e03472729
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lodimk2/chai
  tools:
  - CHAI
  - CHAI-SNF
  - R
  - SAME-clustering
  - SNFtool
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# r-package-scripting

## Summary

Implement and run CHAI's consensus clustering as an open source R package: wrapper functions (CHAI_AvgSim, CHAI_SNF) operate on a SingleCellExperiment object with counts and logcounts and aggregate seven scRNAseq clustering algorithms into a consensus assignment.

## When to use

Use when scripting an R-based consensus clustering pipeline for scRNAseq data packaged as a SingleCellExperiment object, or when extending CHAI with custom clustering algorithms via its R API.

## When NOT to use

- Input data are not in R SingleCellExperiment format (e.g. raw FASTQ, mzML, or a Python AnnData object) — the CHAI wrappers require counts and logcounts assays
- The goal is to run a single clustering method rather than aggregate results from seven algorithms into a consensus
- Data are other formats such as FPKM, CPM and RPKM, which the analogous SAMEclustering package no longer accepts — use raw count data

## Inputs

- SingleCellExperiment object with counts and logcounts assays (e.g. baron_mouse_1 count matrix)
- Candidate best_k integer for Spectral Clustering
- eval flag (TRUE to pick best_k by silhouette score, FALSE to use the supplied best_k)

## Outputs

- SingleCellExperiment annotated with consensus cluster assignments from CHAI-AvgSim or CHAI-SNF
- Evaluated best_k based on maximum average silhouette score over the range 2 to k+1
- Aggregated similarity matrix (AvgSim matrix or fused snf_matrix, m x m with entries in [0,1])

## How to apply

Install the package with devtools::install_github("lodimk2/chai") and load dependencies (chai, scSHC, RaceID, SC3, SingleCellExperiment, CHOIR). Build a SingleCellExperiment with counts and logcounts assays (e.g. via scuttle::logNormCounts on baron_mouse_1). Call the CHAI_AvgSim or CHAI_SNF wrapper with a candidate best_k; when eval=TRUE the package determines best_k for Spectral Clustering by silhouette score instead of using the supplied value. Internally the pipeline runs the seven individual algorithms (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC), builds a binary similarity matrix per algorithm (1 if two cells share a cluster assignment, else 0), and aggregates them either by average similarity or via the SNFtool SNF algorithm before Spectral Clustering. The wrapper returns the SingleCellExperiment annotated with consensus cluster assignments, and users may add their own clustering methods or down-select the ones used for aggregation.

## Related tools

- **CHAI** (The R package being scripted; provides CHAI_AvgSim and CHAI_SNF wrapper functions for consensus clustering) — https://github.com/lodimk2/chai
- **R** (Language in which the CHAI package is written and installed via devtools::install_github)
- **SNFtool** (Supplies the SNF algorithm applied across the seven binary similarity matrices in CHAI-SNF)
- **SAME-clustering** (Alternative R cluster-ensemble package used as the comparison baseline; has a bug preventing SC3 from running) — https://github.com/yycunc/SAMEclustering

## Examples

```
devtools::install_github("lodimk2/chai"); sce <- CHAI_SNF(sce, 15, eval = TRUE)
```

## Evaluation signals

- Package installs cleanly via devtools::install_github("lodimk2/chai") and library(chai) loads without missing scSHC/RaceID/SC3/CHOIR dependencies
- The fused SNF matrix is m x m with entries in [0,1], and every cell receives exactly one consensus cluster label
- With eval=TRUE, the chosen best_k corresponds to the maximum average silhouette score over the evaluated range 2 to k+1
- Consensus assignments are present in the returned SingleCellExperiment for all cells after CHAI_AvgSim or CHAI_SNF completes
- Custom methods added to the pipeline produce their own binary similarity matrix entries (same cluster = 1, different cluster = 0) before aggregation

## Limitations

- An immediate limitation of CHAI is that it is not currently possible to select an ideal set of algorithms to be used in the final consensus; current recommendation is to include as many algorithms as possible
- Individual algorithms such as Spectrum often display subzero and negative ARI (poor-performing component), which can propagate into the consensus
- Data must be supplied as a SingleCellExperiment object with counts and logcounts defined, per the README quick start

## Evidence

- [abstract] CHAI is available as an open source R package on GitHub: "CHAI is available as an open source R package on GitHub"
- [methods] The package is written in R and is available for installation on GitHub: "The package is written in R and is available for installation on GitHub at https://github.com/lodimk2/chai"
- [readme] Data should be in the form of a SingleCellExperiment object, with counts and logcounts defined.: [evidence span withheld — anchor status: external_doc]
- [readme] If "eval" is set to TRUE, CHAI will evaluate the best_k for Spectral Clustering using silhouette score.: "If "eval" is set to TRUE, CHAI will evaluate the best_k for Spectral Clustering using silhouette score."
- [abstract] users may add their own clustering methods to the pipeline, or down-select just the ones they want to use for the clustering aggregation: "users may add their own clustering methods to the pipeline, or down-select just the ones they want to use for the clustering aggregation"
- [discussion] An immediate limitation of CHAI is that it is not currently possible to select an ideal set of algorithms to be used in the final consensus: "An immediate limitation of CHAI is that it is not currently possible to select an ideal set of algorithms to be used in the final consensus"
