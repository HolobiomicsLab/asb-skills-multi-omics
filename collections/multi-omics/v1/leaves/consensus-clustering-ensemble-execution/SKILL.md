---
name: consensus-clustering-ensemble-execution
description: Use when single-cell RNA-seq data require a consensus cell-type clustering
  that ensembles multiple algorithms (e.g., Seurat-Louvain, SC3, Spectrum, scSHC)
  instead of trusting one method's assignments.
when_to_use_negative:
- When a single well-validated clustering result is sufficient and the dataset is
  small/simple — consensus aggregation of seven algorithms carries substantial compute
  cost (e.g., 68K-cell Zheng68K required subsampling to 1000–5000 cells).
- When the goal is multi-omic integration via spatial transcriptomics — that variant
  is handled by CHAI-SNF-ST, a separate extension, not the base AvgSim/SNF ensemble.
- When replicating SAME-clustering comparisons — note SC3 does not run in the SAMEclustering
  package due to a bug, so component sets differ between the two frameworks.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CHAI
  role: R package providing CHAI_AvgSim and CHAI_SNF wrapper functions that run the
    seven constituent clusterers, build binary similarity matrices, aggregate them,
    and perform Spectral Clustering
  repo: https://github.com/lodimk2/chai
- name: Seurat
  role: Constituent clustering algorithms Seurat-Louvain and Seurat-SLC within the
    CHAI ensemble
  repo: https://github.com/lodimk2/chai
- name: SC3
  role: Constituent consensus-clustering method included in the CHAI pipeline
  repo: https://github.com/lodimk2/chai
- name: Spectrum
  role: Constituent clustering algorithm in the CHAI ensemble; known to be a poor
    performer with subzero negative ARI
  repo: https://github.com/lodimk2/chai
- name: scSHC
  role: Constituent statistical-significance-testing clustering algorithm in the CHAI
    ensemble
  repo: https://github.com/lodimk2/chai
- name: CHOIR
  role: Constituent clustering algorithm using hierarchical split hypothesis testing,
    part of the CHAI ensemble
  repo: https://github.com/lodimk2/chai
- name: SAMEclustering
  role: Alternative consensus clustering method used as benchmark; CHAI outperforms
    it, though SC3 is bugged in its package
  repo: https://github.com/yycunc/SAMEclustering
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:2ce0e4bf90e57748381e04edd5eca6f646f66064ca9448d92c0ad5559e001912
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
  - SAMEclustering
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
yes_rate: 0.9411764705882353
n_claims: 17
---

# consensus-clustering-ensemble-execution

## Summary

Run CHAI, a wisdom-of-crowds consensus clustering framework for scRNAseq that converts clustering assignments from seven algorithms into binary similarity matrices and aggregates them via average similarity (CHAI-AvgSim) or Similarity Network Fusion (CHAI-SNF) followed by Spectral Clustering.

## When to use

Use when single-cell RNA-seq data require a consensus cell-type clustering that ensembles multiple algorithms (e.g., Seurat-Louvain, SC3, Spectrum, scSHC) instead of trusting one method's assignments.

## When NOT to use

- When a single well-validated clustering result is sufficient and the dataset is small/simple — consensus aggregation of seven algorithms carries substantial compute cost (e.g., 68K-cell Zheng68K required subsampling to 1000–5000 cells).
- When the goal is multi-omic integration via spatial transcriptomics — that variant is handled by CHAI-SNF-ST, a separate extension, not the base AvgSim/SNF ensemble.
- When replicating SAME-clustering comparisons — note SC3 does not run in the SAMEclustering package due to a bug, so component sets differ between the two frameworks.

## Inputs

- SingleCellExperiment object with counts and logcounts assays (e.g., Baron Mouse 1 dataset baron_mouse_1)
- best_k value for Spectral Clustering (used when eval=FALSE, otherwise silhouette-selected)
- Optionally, user-supplied additional clustering algorithms or a down-selected subset of the seven constituent methods

## Outputs

- Binary similarity matrix per constituent clustering algorithm (same cluster = 1, different cluster = 0)
- Average Similarity (AvgSim) matrix or SNF-integrated matrix
- Consensus per-cell cluster assignments from Spectral Clustering on the integrated matrix
- Selected best_k determined by optimal silhouette score

## How to apply

Run the seven CHAI component clustering algorithms (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC) on a SingleCellExperiment object with counts and logcounts defined, and convert each clustering assignment into a binary similarity matrix assigning 1 when two cells share a cluster and 0 otherwise. Aggregate the matrices either by element-wise averaging (CHAI-AvgSim) or with the SNF algorithm from the SNFtool package with default parameters (CHAI-SNF). Run Spectral Clustering on the integrated matrix; when the true number of clusters is unknown, determine best_k by evaluating the optimal silhouette score over the range 2 to k+1, where k is the expected number of clusters. The input data should follow the README format: a SingleCellExperiment object created from the count matrix (e.g., baron_mouse_1) with logcounts added via scuttle::logNormCounts. Because individual algorithms vary in performance (e.g., Spectrum often shows subzero negative ARI), the authors recommend including as many algorithms as possible in the consensus.

## Related tools

- **CHAI** (R package providing CHAI_AvgSim and CHAI_SNF wrapper functions that run the seven constituent clusterers, build binary similarity matrices, aggregate them, and perform Spectral Clustering) — https://github.com/lodimk2/chai
- **Seurat** (Constituent clustering algorithms Seurat-Louvain and Seurat-SLC within the CHAI ensemble) — https://github.com/lodimk2/chai
- **SC3** (Constituent consensus-clustering method included in the CHAI pipeline) — https://github.com/lodimk2/chai
- **Spectrum** (Constituent clustering algorithm in the CHAI ensemble; known to be a poor performer with subzero negative ARI) — https://github.com/lodimk2/chai
- **scSHC** (Constituent statistical-significance-testing clustering algorithm in the CHAI ensemble) — https://github.com/lodimk2/chai
- **CHOIR** (Constituent clustering algorithm using hierarchical split hypothesis testing, part of the CHAI ensemble) — https://github.com/lodimk2/chai
- **SAMEclustering** (Alternative consensus clustering method used as benchmark; CHAI outperforms it, though SC3 is bugged in its package) — https://github.com/yycunc/SAMEclustering

## Examples

```
devtools::install_github("lodimk2/chai"); sce <- SingleCellExperiment(assays=list(counts=as.matrix(baron_mouse_1))); sce <- scuttle::logNormCounts(sce); sce <- CHAI_AvgSim(sce, best_k=15, eval=TRUE)
```

## Evaluation signals

- The SingleCellExperiment object returned by CHAI_AvgSim/CHAI_SNF contains consensus cluster assignments for every cell in the input (one assignment per cell).
- When eval=TRUE, the selected best_k matches the k maximizing the average silhouette score over the range 2 to k+1 given the true number of clusters.
- ARI of CHAI-AvgSim and CHAI-SNF against ground-truth labels exceeds that of the individual constituent algorithms and of SAME-clustering on benchmark datasets.
- Binary similarity matrices are correctly constructed: symmetric, diagonal of 1, entries 1 iff two cells share a constituent cluster assignment.
- In robustness checks on Zheng68K subsamples (1000/2500/5000 cells, simple vs challenging groups), CHAI-SNF shows the smallest simple-vs-challenging ARI percent difference, confirming its robustness to cluster-complexity changes.

## Limitations

- An immediate limitation of CHAI is that it is not currently possible to select an ideal set of algorithms to be used in the final consensus; the current recommendation is to include as many algorithms as possible.
- Individual clustering algorithms demonstrate large variation in performance (e.g., Spectrum often displays subzero and negative ARI), which can affect the ensemble.
- Further evaluation remains to be done on the best algorithms to use in the consensus pipeline for particular dataset conditions.
- CHAI performs very poorly on some datasets (e.g., the SC-Mixology-Dropseq dataset) despite strong performance on others (e.g., Zeisel mouse brain).

## Evidence

- [intro] CHAI-AvgSim is performed by aggregating all clustering assignments with an average similarity matrix, and performing Spectral Clustering: "CHAI-AvgSim is performed by aggregating all clustering assignments with an average similarity matrix, and performing Spectral Clustering"
- [methods] If two cells have the same clustering assignment, assign a value of 1 to a binary similarity matrix corresponding to the two cells.: "If two cells have the same clustering assignment, assign a value of 1 to a binary similarity matrix corresponding to the two cells."
- [methods] we conduct a silhouette score evaluation in range 2 to k + 1, with k being the true number of clusters present in the dataset: [evidence span withheld — anchor status: not_found]
- [intro] CHAI contains seven state-of-the-art scRNAseq clustering algorithms (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, and scSHC) and is available as an R package: "CHAI contains seven state-of-the-art scRNAseq clustering algorithms (Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, and scSHC) and is available as an R package"
- [readme] Data should be in the form of a SingleCellExperiment object, with counts and logcounts defined.: [evidence span withheld — anchor status: external_doc]
