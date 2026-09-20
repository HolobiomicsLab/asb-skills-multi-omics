---
name: scrna-seq-clustering-benchmarking
description: Use when comparing clustering methods on single-cell RNA-seq datasets
  with known cell-type labels, e.g. Baron, Zeisel, Zheng68K, or SC-Mixology, to decide
  which method performs .
when_to_use_negative:
- Input has no ground-truth cell-type labels, so ARI/NMI against a reference cannot
  be computed (unsupervised quality assessment is out of scope here)
- The goal is multiomic or spatial transcriptomics integration rather than scRNAseq
  clustering benchmarking (CHAI-SNF-ST / spatial workflows are a separate skill)
- The dataset format is FPKM/CPM/RPKM rather than count data, which SAMEclustering
  no longer accepts
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CHAI
  role: R package providing the seven default clustering algorithms and the CHAI_AvgSim/CHAI_SNF
    consensus wrappers
  repo: https://github.com/lodimk2/chai
- name: SNFtool
  role: Provides the SNF algorithm used with default parameters to build the CHAI-SNF
    integrated similarity matrix
- name: SAMEclustering
  role: Consensus clustering baseline (SC3, CIDR, Seurat, t-SNE + k-means, SIMLR inputs,
    EM ensemble) for ARI/NMI comparison
  repo: https://github.com/yycunc/SAMEclustering
- name: sc_mixology
  role: Source of the SC-Mixology benchmarking datasets (HCC827, H1975, H2228 cell
    lines; 10X/CEL-seq2/Drop-seq) with ground-truth labels
  repo: https://github.com/LuyiTian/sc_mixology
- name: R
  role: Implementation language; CHAI is installed via devtools::install_github
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:b5c0b2cb3f7762b4d388bbd330457f7c3b470fecb0ce0f435b4fbdf1759daf91
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lodimk2/chai
  tools:
  - CHAI
  - CHAI-AvgSim
  - CHAI-SNF
  - R
  - SAME-clustering
  - SAMEclustering
  - SC3
  - SNFtool
  - Seurat
  - Spectrum
  - scSHC
  - sc_mixology
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# scrna-seq-clustering-benchmarking

## Summary

Benchmark scRNAseq clustering and consensus clustering methods (CHAI-AvgSim, CHAI-SNF, SAME-clustering, and seven individual algorithms) across public datasets by computing ARI and NMI against ground-truth cell-type labels.

## When to use

Use when comparing clustering methods on single-cell RNA-seq datasets with known cell-type labels, e.g. Baron, Zeisel, Zheng68K, or SC-Mixology, to decide which method performs best.

## When NOT to use

- Input has no ground-truth cell-type labels, so ARI/NMI against a reference cannot be computed (unsupervised quality assessment is out of scope here)
- The goal is multiomic or spatial transcriptomics integration rather than scRNAseq clustering benchmarking (CHAI-SNF-ST / spatial workflows are a separate skill)
- The dataset format is FPKM/CPM/RPKM rather than count data, which SAMEclustering no longer accepts

## Inputs

- scRNAseq count matrices as SingleCellExperiment objects (counts and logcounts), e.g. baron_mouse_1, Zheng68K PBMC, Zeisel mouse brain, SC-Mixology (CEL-seq2/10X/Dropseq subsets)
- Ground-truth cell-type label vectors per dataset
- Per-method clustering assignment tables (Cell ID / cluster label per cell)

## Outputs

- benchmark_ari_results.csv: per-dataset ARI and NMI for each method (methods x datasets, plus per-method rank statistics)
- Consensus cluster assignments from CHAI-AvgSim / CHAI-SNF Spectral Clustering
- Binary similarity matrices per algorithm, Average Similarity matrix, and SNF-integrated matrix

## How to apply

Load each benchmarking dataset (SingleCellExperiment with counts/logcounts) and run the seven CHAI algorithms (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC) plus the SAME-clustering baseline. Convert each method's Cell ID / clustering assignment table into an m x m binary similarity matrix (same cluster = 1, different = 0), then build CHAI-AvgSim by element-wise averaging and CHAI-SNF via the SNF algorithm with SNFtool defaults. Choose best_k for Spectral Clustering by evaluating the silhouette score in the range 2 to k+1 (with k the true cluster number) or setting eval=TRUE to have CHAI select it. Finally compute ARI and NMI between predicted clusters and ground truth per dataset, and assemble a methods x datasets benchmark table with per-method rank statistics.

## Related tools

- **CHAI** (R package providing the seven default clustering algorithms and the CHAI_AvgSim/CHAI_SNF consensus wrappers) — https://github.com/lodimk2/chai
- **SNFtool** (Provides the SNF algorithm used with default parameters to build the CHAI-SNF integrated similarity matrix)
- **SAMEclustering** (Consensus clustering baseline (SC3, CIDR, Seurat, t-SNE + k-means, SIMLR inputs, EM ensemble) for ARI/NMI comparison) — https://github.com/yycunc/SAMEclustering
- **sc_mixology** (Source of the SC-Mixology benchmarking datasets (HCC827, H1975, H2228 cell lines; 10X/CEL-seq2/Drop-seq) with ground-truth labels) — https://github.com/LuyiTian/sc_mixology
- **R** (Implementation language; CHAI is installed via devtools::install_github)

## Examples

```
devtools::install_github("lodimk2/chai"); sce <- SingleCellExperiment(assays=list(counts=as.matrix(baron_mouse_1))); sce <- scuttle::logNormCounts(sce); sce <- CHAI_AvgSim(sce, 15, eval=TRUE)
```

## Evaluation signals

- ARI and NMI computed between predicted clusters and ground-truth labels for every method x dataset combination; CHAI-AvgSim and CHAI-SNF should outperform SAME-clustering in aggregate ranks
- Binary similarity matrices must be symmetric m x m with entries 1 only for cell pairs sharing a cluster
- best_k selection: silhouette score evaluated in range 2 to k+1; running with eval=TRUE should reproduce this selection
- SC3 will not run within the SAMEclustering package (known bug, issue #4), so baseline results should exclude SC3 where the bug applies
- Spectrum often shows subzero/negative ARI, so near-zero or negative Spectrum ARI values are expected rather than an error

## Limitations

- There is no method yet to select an ideal subset of algorithms for the CHAI consensus; the authors recommend including as many algorithms as possible
- Individual clustering algorithms show large variation in performance across datasets, and further evaluation is needed on the best algorithm set for particular dataset conditions
- SC3 cannot run within the current SAMEclustering package due to a bug, so SC3 is in CHAI's pipeline but not SAME-clustering's in all evaluations, complicating direct comparison

## Evidence

- [intro] CHAI contains seven state-of-the-art scRNAseq clustering algorithms (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, and scSHC) and is available as an R package: "CHAI contains seven state-of-the-art scRNAseq clustering algorithms (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, and scSHC) and is available as an R package"
- [methods] silhouette score evaluation in range 2 to k + 1: [evidence span withheld — anchor status: not_found]
- [methods] If two cells have the same clustering assignment, assign a value of 1 to a binary similarity matrix corresponding to the two cells.: "If two cells have the same clustering assignment, assign a value of 1 to a binary similarity matrix corresponding to the two cells."
- [abstract] both CHAI methods outperform the most recent consensus clustering method, SAME-clustering: "both CHAI methods outperform the most recent consensus clustering method, SAME-clustering"
- [discussion] SC3 does not run in its package due to a bug (see: https://github.com/yycunc/SAMEclustering/issues/4: "SC3 does not run in its package due to a bug (see: https://github.com/yycunc/SAMEclustering/issues/4"
- [readme] You may install CHAI using devtools.: [evidence span withheld — anchor status: external_doc]
- [readme] Data should be in the form of a SingleCellExperiment object, with counts and logcounts defined.: [evidence span withheld — anchor status: external_doc]
