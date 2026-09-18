---
name: consensus-clustering-baseline-comparison
description: Use when evaluating whether a new consensus/ensemble clustering method
  for scRNAseq outperforms existing individual algorithms (Seurat-Louvain, Seurat-SLC,
  CHOIR, RACEID, SC3, Spectrum, scSHC) and the SAME-clustering ensemble, using ARI/NMI
  against known cell-type labels.
when_to_use_negative:
- The input is spatial transcriptomics data with a multiomic integration goal — the
  article handles that with the CHAI-SNF-ST extension (GraphST comparison), not this
  plain scRNAseq baseline comparison
- Ground-truth cell-type labels are unavailable, since ARI/NMI against true labels
  cannot be computed
- Only a single clustering method's output needs evaluation — consensus ensemble benchmarking
  adds unnecessary overhead
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CHAI
  role: R package providing CHAI-AvgSim and CHAI-SNF wrapper functions that run the
    seven algorithms, build binary similarity matrices, aggregate via average similarity
    or SNF, and run Spectral Clustering
  repo: https://github.com/lodimk2/chai
- name: SAME-clustering
  role: Consensus clustering baseline that aggregates SC3, CIDR, Seurat, t-SNE + k-means
    and SIMLR solutions via mixture-model EM; compared against CHAI via ARI
  repo: https://github.com/yycunc/SAMEclustering
- name: SNFtool
  role: Provides the Similarity Network Fusion algorithm applied across the binary
    similarity matrices to build the CHAI-SNF integrated matrix
- name: SC3
  role: One of CHAI's seven constituent clustering algorithms; also used within SAME-clustering's
    individual_clustering
- name: Spectrum
  role: One of CHAI's seven constituent clustering algorithms; noted for poor performance
    with subzero and negative ARI
- name: scSHC
  role: One of CHAI's seven constituent clustering algorithms, using statistical significance
    testing for cluster number
- name: CHOIR
  role: One of CHAI's seven constituent clustering algorithms, using hierarchical
    split hypothesis testing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:1dae308ad6dc0f5f75c47c4ac4543594730c32a5a7f332acf677a0ea9f89098a
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
  - SAME-clustering
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
yes_rate: 0.8125
n_claims: 16
---

# consensus-clustering-baseline-comparison

## Summary

Benchmark the two CHAI consensus clustering ensembles (CHAI-AvgSim and CHAI-SNF), built from seven scRNAseq clustering algorithms via binary similarity matrices, against the SAME-clustering consensus baseline using ARI and NMI on ground-truth-labeled benchmark datasets.

## When to use

Use when evaluating whether a new consensus/ensemble clustering method for scRNAseq outperforms existing individual algorithms (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC) and the SAME-clustering ensemble, using ARI/NMI against known cell-type labels.

## When NOT to use

- The input is spatial transcriptomics data with a multiomic integration goal — the article handles that with the CHAI-SNF-ST extension (GraphST comparison), not this plain scRNAseq baseline comparison
- Ground-truth cell-type labels are unavailable, since ARI/NMI against true labels cannot be computed
- Only a single clustering method's output needs evaluation — consensus ensemble benchmarking adds unnecessary overhead

## Inputs

- scRNAseq count matrices as SingleCellExperiment objects with counts and logcounts assays (e.g., baron_mouse_1)
- Ground-truth cell-type label vectors for each benchmark dataset (e.g., data_SAME$Zheng.celltype)
- Per-algorithm Cell ID / clustering assignment tables (m cells x methods)
- SAME-clustering individual clustering result matrix (M methods x N cells)

## Outputs

- Per-dataset ARI and NMI values for each individual algorithm, CHAI-AvgSim, CHAI-SNF, and SAME-clustering
- Methods x datasets benchmark table (e.g., benchmark_ari_results.csv) with per-method rank statistics
- Final consensus cluster assignment vectors from Spectral Clustering on the AvgSim or SNF matrix

## How to apply

Run each of CHAI's seven default clustering algorithms (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC) on every benchmark dataset (e.g., Baron Mouse_1, Muraro, SC-Mixology including the Dropseq subset, Zeisel mouse brain, Zheng68K PBMC). Convert each algorithm's Cell ID / clustering assignment table into an m x m binary similarity matrix, scoring 1 for cell pairs sharing a cluster and 0 otherwise. Build CHAI-AvgSim by element-wise averaging of these matrices and CHAI-SNF by applying the SNF algorithm (SNFtool default parameters); for both, select k by maximum average silhouette score evaluated in the range 2 to k+1, then run Spectral Clustering on the integrated matrix to obtain final cell identities. Run the SAME-clustering ensemble (SC3, CIDR, Seurat, t-SNE + k-means, SIMLR, with a maximally diverse subset of four solutions by pairwise ARI) on the same datasets as the consensus baseline. Compute ARI and NMI between each method's predicted clusters and ground-truth labels per dataset, then assemble a methods x datasets benchmark table.

## Related tools

- **CHAI** (R package providing CHAI-AvgSim and CHAI-SNF wrapper functions that run the seven algorithms, build binary similarity matrices, aggregate via average similarity or SNF, and run Spectral Clustering) — https://github.com/lodimk2/chai
- **SAME-clustering** (Consensus clustering baseline that aggregates SC3, CIDR, Seurat, t-SNE + k-means and SIMLR solutions via mixture-model EM; compared against CHAI via ARI) — https://github.com/yycunc/SAMEclustering
- **SNFtool** (Provides the Similarity Network Fusion algorithm applied across the binary similarity matrices to build the CHAI-SNF integrated matrix)
- **SC3** (One of CHAI's seven constituent clustering algorithms; also used within SAME-clustering's individual_clustering)
- **Spectrum** (One of CHAI's seven constituent clustering algorithms; noted for poor performance with subzero and negative ARI)
- **scSHC** (One of CHAI's seven constituent clustering algorithms, using statistical significance testing for cluster number)
- **CHOIR** (One of CHAI's seven constituent clustering algorithms, using hierarchical split hypothesis testing)

## Examples

```
sce <- CHAI_AvgSim(sce, best_k, eval = TRUE)  # after devtools::install_github("lodimk2/chai"); adjustRandIndex via mclust for ARI vs ground truth
```

## Evaluation signals

- Both CHAI-AvgSim and CHAI-SNF should show higher ARI and NMI than SAME-clustering on most benchmark datasets, per the article's headline finding
- Per-dataset ARI/NMI values must fall within [-1, 1] and [0, 1] respectively; Spectrum often displays subzero and negative ARI, which flags a poor-performing component algorithm
- The integrated similarity matrices must be m x m binary-derived matrices (AvgSim entries in [0,1]); cluster number k is chosen at the maximum average silhouette score in the range 2 to k+1
- SAME-clustering's ensemble output uses the BIC-cluster solution (cluster.ensemble$BICcluster) for the ARI comparison via adjustedRandIndex
- Reproducibility check: SC3 does not run within the current SAME-clustering package due to a bug (github.com/yycunc/SAMEclustering/issues/4), so SC3 inclusion differs between CHAI's and SAME-clustering's pipelines

## Limitations

- SC3 does not run in the SAME-clustering package due to a bug, and SC3 is included in CHAI's pipeline but not in SAME-clustering's in all evaluations — the comparison is not perfectly apples-to-apples
- There is no current method to select an ideal subset of algorithms for the CHAI consensus; the recommendation is to include as many algorithms as possible pending community best practices
- Individual clustering algorithms demonstrate large variation in performance across datasets (e.g., strong on Zeisel mouse brain but very poor on SC-Mixology-Dropseq), and further evaluation of the best algorithm subset for particular dataset conditions remains to be done

## Evidence

- [abstract] CHAI presents two competing methods which aggregate the clustering results from seven state-of-the-art clustering methods: CHAI-AvgSim and CHAI-SNF: "CHAI presents two competing methods which aggregate the clustering results from seven state-of-the-art clustering methods: CHAI-AvgSim and CHAI-SNF"
- [abstract] both CHAI methods outperform SAME-clustering: "both CHAI methods outperform the most recent consensus clustering method, SAME-clustering"
- [methods] silhouette score evaluation in range 2 to k + 1: [evidence span withheld — anchor status: not_found]
- [methods] binary similarity matrix assignment rule: "If two cells have the same clustering assignment, assign a value of 1 to a binary similarity matrix corresponding to the two cells."
- [readme] SAME-clustering uses five methods and a diverse subset of four: [evidence span withheld — anchor status: not_found]
- [discussion] SC3 bug in SAME-clustering: "SC3 does not run in its package due to a bug (see: https://github.com/yycunc/SAMEclustering/issues/4"
