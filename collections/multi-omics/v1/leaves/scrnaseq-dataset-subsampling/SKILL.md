---
name: scrnaseq-dataset-subsampling
description: Use when you must test whether a scRNAseq clustering method's performance
  (e.g., ARI on Zheng68K) is robust to dataset size and cell-type complexity, by benchmarking
  on controlled subsampled datasets.
when_to_use_negative:
- The dataset is already small enough that subsampling would discard meaningful signal,
  or the analysis goal is absolute benchmark accuracy rather than robustness to size/complexity
  variation.
- Ground-truth cell-type labels are unavailable, since the simple-vs-challenging group
  construction and ARI computation both depend on known labels.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CHAI
  role: R package (CHAI-AvgSim / CHAI-SNF) run on each subsample to assess robustness
    of the consensus clustering
  repo: https://github.com/lodimk2/chai
- name: Seurat
  role: Constituent clustering algorithm (Seurat-Louvain, Seurat-SLC) run per subsample
    before aggregation
- name: SC3
  role: Constituent clustering algorithm run per subsample before aggregation
- name: Spectrum
  role: Constituent clustering algorithm run per subsample before aggregation
- name: scSHC
  role: Constituent clustering algorithm run per subsample before aggregation
- name: CHOIR
  role: Constituent clustering algorithm run per subsample before aggregation
- name: RaceID
  role: Constituent clustering algorithm (RACEID) run per subsample before aggregation
- name: R
  role: Implementation language for subsampling, clustering, and ARI computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:0ead26bdb7c8c21a44bc578072f536800ed690f0284f22c33ebe3c1706ac2520
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lodimk2/chai
  tools:
  - CHAI
  - CHAI-AvgSim
  - CHAI-SNF
  - CHOIR
  - R
  - RaceID
  - SC3
  - Seurat
  - Spectrum
  - scSHC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# scrnaseq-dataset-subsampling

## Summary

Systematically subsample a large single-cell RNA-seq dataset (e.g., Zheng68K PBMC) into smaller datasets of varying size (e.g., 1000, 2500, 5000 cells) and cluster-complexity (equally sized 'simple' vs randomly selected 'challenging' cell-type groups), then re-run a clustering pipeline on each subsample to assess robustness to dataset size and complexity.

## When to use

Use when you must test whether a scRNAseq clustering method's performance (e.g., ARI on Zheng68K) is robust to dataset size and cell-type complexity, by benchmarking on controlled subsampled datasets.

## When NOT to use

- The dataset is already small enough that subsampling would discard meaningful signal, or the analysis goal is absolute benchmark accuracy rather than robustness to size/complexity variation.
- Ground-truth cell-type labels are unavailable, since the simple-vs-challenging group construction and ARI computation both depend on known labels.

## Inputs

- Large scRNAseq count matrix with ground-truth cell-type labels (e.g., Zheng68K PBMC dataset)
- SingleCellExperiment object with counts and logcounts assays
- Subsample specification: sizes (1000, 2500, 5000 cells) and complexity conditions (simple = five equally sized cell-type groups; challenging = randomly selected groups)

## Outputs

- Six subsampled SingleCellExperiment datasets (three sizes x two complexity conditions)
- Clustering assignments per method per subsample
- Binary similarity matrices per clustering run (same cluster = 1, different cluster = 0)
- ARI values per subsample and per ensemble method (CHAI-AvgSim, CHAI-SNF)
- Summary table of simple-vs-challenging ARI percent differences (e.g., ari_simple_vs_challenging.csv)

## How to apply

Load a large scRNAseq dataset with ground-truth labels as a SingleCellExperiment object (counts plus logcounts via logNormCounts), e.g., the Zheng68K PBMC dataset used in the CHAI paper. Generate subsamples across multiple sizes (1000, 2500, 5000 cells) and two complexity regimes: 'simple' subsamples with equally sized cell-type groups and 'challenging' subsamples with randomly selected cell-type groups, producing one subsample per size-per-condition combination (six datasets). Run the same clustering pipeline (e.g., the seven CHAI constituents: Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC, followed by CHAI-AvgSim or CHAI-SNF aggregation) on each subsample, selecting k for Spectral Clustering by maximizing the silhouette score over the range 2 to k+1 where k is the true number of clusters. Score each result with Adjusted Rand Index (ARI) against ground-truth labels and compare ARI between simple and challenging conditions at each size (e.g., absolute percent difference) to quantify robustness to complexity changes.

## Related tools

- **CHAI** (R package (CHAI-AvgSim / CHAI-SNF) run on each subsample to assess robustness of the consensus clustering) — https://github.com/lodimk2/chai
- **Seurat** (Constituent clustering algorithm (Seurat-Louvain, Seurat-SLC) run per subsample before aggregation)
- **SC3** (Constituent clustering algorithm run per subsample before aggregation)
- **Spectrum** (Constituent clustering algorithm run per subsample before aggregation)
- **scSHC** (Constituent clustering algorithm run per subsample before aggregation)
- **CHOIR** (Constituent clustering algorithm run per subsample before aggregation)
- **RaceID** (Constituent clustering algorithm (RACEID) run per subsample before aggregation)
- **R** (Implementation language for subsampling, clustering, and ARI computation)

## Examples

```
devtools::install_github("lodimk2/chai"); sce <- SingleCellExperiment(assays=list(counts=as.matrix(subsampled_counts))); sce <- scuttle::logNormCounts(sce); sce <- CHAI_SNF(sce, best_k, eval=TRUE)
```

## Evaluation signals

- Six subsamples are produced: three sizes (1000, 2500, 5000 cells) x two conditions (simple, challenging).
- Spectral Clustering k is selected by maximizing the average silhouette score over the range 2 to k+1, with k equal to the true number of clusters.
- ARI is computed against ground-truth labels for every subsample and every method, giving directly comparable values across conditions.
- Validation from the source task: CHAI-SNF exhibits the smallest simple-vs-challenging percent difference in ARI (including performance increases on challenging sets), reproducing the paper's reported robustness of CHAI-SNF.
- Subsampled data must remain a valid SingleCellExperiment with counts and logcounts defined, matching CHAI's input requirements.

## Limitations

- Individual clustering algorithms demonstrate large variation in performance; Spectrum often displays subzero and negative ARI, so per-method results on subsamples may be dominated by one poor-performing component.
- It is not currently possible to select an ideal set of algorithms to be used in the final consensus, so subsample robustness results reflect the full seven-method panel.
- Further evaluation remains to be done on the best algorithms to use in the consensus pipeline for particular dataset conditions.

## Evidence

- [methods] we took advantage of the size and complexity in the Zheng68K PBMC dataset to create subsampled datasets: "we took advantage of the size and complexity in the Zheng68K PBMC dataset to create subsampled datasets"
- [methods] silhouette k selection: [evidence span withheld — anchor status: not_found]
- [intro] binary similarity construction: "If two cells are assigned the same cluster, assign a score of 1."
- [readme] CHAI input format: [evidence span withheld — anchor status: external_doc]
