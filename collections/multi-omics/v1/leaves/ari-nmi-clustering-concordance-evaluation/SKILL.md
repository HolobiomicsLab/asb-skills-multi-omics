---
name: ari-nmi-clustering-concordance-evaluation
description: Use when predicted cluster labels from one or more clustering methods
  must be scored against known cell-type labels (e.g., building a methods x datasets
  benchmark table of ARI/NMI across Baron, Zeisel, SC-Mixology, or Zheng68K datasets).
when_to_use_negative:
- Ground-truth cell-type labels are unavailable — ARI/NMI require reference labels
  and cannot be computed (use internal metrics like silhouette score instead)
- Comparing clusterings of different cell sets — ARI/NMI require the two label vectors
  to be defined over the identical set of cells
- Evaluating an unlabeled exploratory clustering where the goal is stability rather
  than concordance with known biology
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: CHAI
  role: Source of consensus cluster assignments (CHAI-AvgSim and CHAI-SNF) being evaluated
    by ARI/NMI against ground-truth labels
  repo: https://github.com/lodimk2/chai
- name: SAMEclustering
  role: Consensus clustering baseline whose ensemble solution is compared to true
    labels via adjustedRandIndex
  repo: https://github.com/yycunc/SAMEclustering
- name: sc_mixology
  role: Provides benchmarking datasets (HCC827, H1975, H2228 cell lines via CEL-seq2,
    Drop-seq, 10X Chromium) with known labels for concordance evaluation
  repo: https://github.com/LuyiTian/sc_mixology
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:7aaea49b4c449e9f6fc25b01922a354c0b09bf5c9a54d7db8e2b0a1be9c9de55
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# ari-nmi-clustering-concordance-evaluation

## Summary

Quantify concordance between predicted scRNAseq cluster assignments and ground-truth cell-type labels using Adjusted Rand Index (ARI) and Normalized Mutual Information (NMI), as done in the CHAI benchmarking study to rank CHAI-AvgSim, CHAI-SNF, seven individual algorithms, and SAME-clustering.

## When to use

Use when predicted cluster labels from one or more clustering methods must be scored against known cell-type labels (e.g., building a methods x datasets benchmark table of ARI/NMI across Baron, Zeisel, SC-Mixology, or Zheng68K datasets).

## When NOT to use

- Ground-truth cell-type labels are unavailable — ARI/NMI require reference labels and cannot be computed (use internal metrics like silhouette score instead)
- Comparing clusterings of different cell sets — ARI/NMI require the two label vectors to be defined over the identical set of cells
- Evaluating an unlabeled exploratory clustering where the goal is stability rather than concordance with known biology

## Inputs

- Vector of predicted cluster assignments per cell (from CHAI-AvgSim, CHAI-SNF, SC3, Seurat, Spectrum, scSHC, CHOIR, RACEID, or SAME-clustering ensemble)
- Vector of ground-truth cell-type labels aligned to the same cell IDs
- Per-dataset clustering result tables (Cell ID / Clustering Assignment)

## Outputs

- Per-dataset ARI and NMI values for each clustering method
- Methods x datasets benchmark table (e.g., benchmark_ari_results.csv) with rank statistics

## How to apply

Run each clustering method (e.g., the seven CHAI algorithms Seurat-Louvain, Seurat-SLC, CHOIR, RACEID, SC3, Spectrum, scSHC, plus CHAI-AvgSim/CHAI-SNF or SAME-clustering) on datasets with ground-truth cell-type labels. For each method-dataset pair, compute ARI and NMI between predicted clusters and true labels; in R, mclust's adjustedRandIndex is the standard ARI implementation as shown in the SAMEclustering README. Assemble the per-dataset scores into a methods x datasets table (e.g., benchmark_ari_results.csv) with per-method rank statistics to judge which consensus approach wins. Interpret negative or subzero ARI as evidence a component method (e.g., Spectrum) performed worse than random on that dataset.

## Related tools

- **CHAI** (Source of consensus cluster assignments (CHAI-AvgSim and CHAI-SNF) being evaluated by ARI/NMI against ground-truth labels) — https://github.com/lodimk2/chai
- **SAMEclustering** (Consensus clustering baseline whose ensemble solution is compared to true labels via adjustedRandIndex) — https://github.com/yycunc/SAMEclustering
- **sc_mixology** (Provides benchmarking datasets (HCC827, H1975, H2228 cell lines via CEL-seq2, Drop-seq, 10X Chromium) with known labels for concordance evaluation) — https://github.com/LuyiTian/sc_mixology

## Examples

```
library(mclust); ari <- adjustedRandIndex(cluster.ensemble$BICcluster, data_SAME$Zheng.celltype)
```

## Evaluation signals

- ARI between predicted clusters and ground-truth labels is in [-1, 1], with values near or below 0 (as seen for Spectrum) indicating worse-than-random agreement
- Every method x dataset cell in the benchmark table has both an ARI and an NMI value; no NAs where labels exist
- Per-method rank statistics computed across datasets are consistent with the per-dataset scores in benchmark_ari_results.csv
- adjustedRandIndex output in R matches the number of cells in the intersection of predicted and true label vectors (length alignment check)

## Limitations

- SC3 does not run within the current SAME-clustering package due to a bug, so SC3 is included in CHAI's pipeline but not in SAME-clustering's in all evaluations, making some comparisons uneven
- Individual clustering algorithms demonstrate large variation in performance, so per-dataset ARI rankings can swing dramatically (e.g., poor on SC-Mixology-Dropseq vs strong on Zeisel mouse brain)
- Spectrum often displays subzero and negative ARI, acting as a poor-performing component in the consensus

## Evidence

- [readme] We can compare the clustering results to the true labels using the ARI. In our implementation, we use the clusters produced using the BIC criterion as our ensemble solution.: [evidence span withheld — anchor status: external_doc]
- [abstract] Both CHAI methods outperform SAME-clustering: "both CHAI methods outperform the most recent consensus clustering method, SAME-clustering"
- [discussion] Spectrum poor performance discussion: [evidence span withheld — anchor status: not_found]
- [discussion] SC3 comparability caveat: [evidence span withheld — anchor status: not_found]
- [readme] adjustedRandIndex usage in SAMEclustering: [evidence span withheld — anchor status: external_doc]
