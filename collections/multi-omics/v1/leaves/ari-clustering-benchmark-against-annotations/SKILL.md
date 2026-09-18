---
name: ari-clustering-benchmark-against-annotations
description: Use when per-spot cluster assignments from one or more spatial transcriptomic
  analysis strategies must be quantitatively ranked against expert/pathologist-annotated
  tissue region labels using ARI, e.g. to reproduce a published strategy ranking.
when_to_use_negative:
- No expert/pathologist-annotated ground-truth labels exist for the sample's spots
  — ARI against annotations cannot be computed and a diagnostic-graph-based strategy
  choice is the intended alternative
- 'The goal is to choose an integration strategy for a new dataset rather than evaluate
  label agreement: the article states ''no single approach performing well in every
  dataset'', so transferring the H1 ARI ranking as a universal winner is inappropriate'
- The data lack per-spot spatial coordinates or an H&E image, so the spatially-aware
  and multimodal stImage strategies being benchmarked cannot be run
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: stImage
  role: R package providing the 12 strategy variants (LoadImageFeature, PreProcessing,
    FindSNNClusters, SpatialAwareProcess, MultiModalIntegration) whose cluster assignments
    are benchmarked by ARI
  repo: https://github.com/YuWang-VUMC/stImage
- name: Seurat
  role: Underlying single-cell toolkit used by stImage for SCT normalization, PCA
    reductions, and SNN clustering
  repo: https://github.com/satijalab/seurat
- name: SpatialPCA
  role: Spatially-aware method whose output (with WNN integration) yields the top-benchmarked
    clustering (ARI 0.62); installed as an stImage dependency
  repo: https://github.com/shangll123/SpatialPCA
- name: BayesSpace
  role: Spatially-aware clustering strategy included in the benchmark (lowest performer,
    ARI ~0.29); installed via Bioconductor
  repo: https://github.com/edward130603/BayesSpace
- name: tidyverse
  role: Used in the stImage methods for data wrangling of count matrices and coordinates
    when assembling the ARI comparison table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:e9ffe2f0e640bd42d1469857c849fdd9a41b96bf0bd2fc96c0b61ab1b54548b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuWang-VUMC/stImage
  tools:
  - BayesSpace
  - R
  - Seurat
  - SpatialPCA
  - stImage
  - tidyverse
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# ari-clustering-benchmark-against-annotations

## Summary

Benchmarks spatial transcriptomic clustering strategies by computing the adjusted Rand index (ARI) of each strategy's cluster assignments against pathologist-annotated tissue-region labels, as done for 12 stImage strategies on the HER2-positive breast cancer H1 sample (SpatialPCA+WNN achieved the top ARI of 0.62).

## When to use

Use when per-spot cluster assignments from one or more spatial transcriptomic analysis strategies must be quantitatively ranked against expert/pathologist-annotated tissue region labels using ARI, e.g. to reproduce a published strategy ranking.

## When NOT to use

- No expert/pathologist-annotated ground-truth labels exist for the sample's spots — ARI against annotations cannot be computed and a diagnostic-graph-based strategy choice is the intended alternative
- The goal is to choose an integration strategy for a new dataset rather than evaluate label agreement: the article states 'no single approach performing well in every dataset', so transferring the H1 ARI ranking as a universal winner is inappropriate
- The data lack per-spot spatial coordinates or an H&E image, so the spatially-aware and multimodal stImage strategies being benchmarked cannot be run

## Inputs

- Gene count matrix H1.tsv (Spatial Transcriptomics platform, HER2 breast cancer patient H1, Zenodo 10.5281/zenodo.4751624)
- H1_labeled_coordinates.tsv (spot coordinates with pathologist-annotated tissue region labels, spot names formatted X<x>x<y>)
- H&E image HE_BT24044_D2.jpg (patch size 200)
- Cluster assignment vectors (6 clusters) from each of the 12 stImage analysis strategies

## Outputs

- HER2_H1_ARI_strategies.csv — adjusted Rand index of every strategy's 6-cluster assignment vs pathologist annotations, enabling ranking comparison to the published benchmark (SpatialPCA+WNN = 0.62)

## How to apply

Run every candidate clustering strategy on the same preprocessed object with an identical cluster count (nCluster=6 for HER2 H1), covering gene-only/image-only baselines, spatially-aware methods (BayesSpace, SpatialPCA, stLearn), and multimodal integrations (WNN, MCIA, tICA, Spectrum) per stImage's FindSNNClusters and MultiModalIntegration workflow. For each strategy, map its spot-level cluster assignment onto the pathologist labels carried in H1_labeled_coordinates.tsv and compute the adjusted Rand index. In this benchmark, the expected ordering was Gene/Image (~0.43-0.44) < SpatialPCA/stLearn (~0.45-0.48) < WNN (0.58) < SpatialPCA+WNN (0.62), with BayesSpace (~0.29) and Spectrum/tICA (~0.40-0.41) as low performers. Before clustering, remove the 21 ring pattern genes mentioned in the original paper, and standardize spot names as X<x>x<y> with patch size 200. Write all ARI values to a results table (HER2_H1_ARI_strategies.csv) so the ranking can be compared to the reported benchmark.

## Related tools

- **stImage** (R package providing the 12 strategy variants (LoadImageFeature, PreProcessing, FindSNNClusters, SpatialAwareProcess, MultiModalIntegration) whose cluster assignments are benchmarked by ARI) — https://github.com/YuWang-VUMC/stImage
- **Seurat** (Underlying single-cell toolkit used by stImage for SCT normalization, PCA reductions, and SNN clustering) — https://github.com/satijalab/seurat
- **SpatialPCA** (Spatially-aware method whose output (with WNN integration) yields the top-benchmarked clustering (ARI 0.62); installed as an stImage dependency) — https://github.com/shangll123/SpatialPCA
- **BayesSpace** (Spatially-aware clustering strategy included in the benchmark (lowest performer, ARI ~0.29); installed via Bioconductor) — https://github.com/edward130603/BayesSpace
- **tidyverse** (Used in the stImage methods for data wrangling of count matrices and coordinates when assembling the ARI comparison table)

## Examples

```
st_obj <- stImage::PreProcessing(countTable, imageFeatures, positionTable, pcaDim_s=20, pcaDim_i=20, pcaDim_c=6); clusters <- stImage::FindSNNClusters(st_obj, nCluster=6, reduction='SCTPCA')
```

## Evaluation signals

- SpatialPCA followed by WNN reaches ARI ≈ 0.62 on HER2 patient H1 with nCluster=6, and the full ordering matches the reported benchmark (Gene/Image ~0.43-0.44 < SpatialPCA/stLearn ~0.45-0.48 < WNN 0.58 < SpatialPCA+WNN 0.62; BayesSpace 0.29; Spectrum/tICA 0.40-0.41)
- Every strategy produces exactly 6 clusters, matching the FindSNNClusters nCluster=6 setting used for the H1 sample
- The number of spot labels in H1_labeled_coordinates.tsv matches the number of spots (X<x>x<y> formatted) in the H1.tsv count matrix after removal of the 21 ring pattern genes
- ARI values for all 12 strategies fall in [0, 1] and HER2_H1_ARI_strategies.csv contains one row per strategy

## Limitations

- The article found 'no single approach performing well in every dataset', so the H1-specific ARI ranking (SpatialPCA+WNN on top) is not guaranteed to transfer to other samples or platforms
- Multi-modal integration strategies in the benchmark are computationally more intensive and algorithm-dependent, though runtime remains manageable on a typical office desktop
- ARI requires a fixed pre-specified cluster count (6 here) matching the annotation granularity; divergent k values invalidate the comparison

## Evidence

- [readme] StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images.: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images."
- [abstract] We demonstrate stImage's effectiveness across multiple datasets, underscoring its ability to guide users toward the most suitable integration strategy using diagnostic graph.: "We demonstrate stImage’s effectiveness across multiple datasets, underscoring its ability to guide users toward the most suitable integration strategy using diagnostic graph."
- [methods] remove the 21 ring pattern genes mentioned in the orginal paper: "remove the 21 ring pattern genes mentioned in the orginal paper"
- [discussion] we recommend applying a spatial aware process followed by modality integration to fully leverage the complementary information: "we recommend applying a spatial aware process followed by modality integration to fully leverage the complementary information"
- [discussion] As we expected, there was no single approach performing well in every dataset.: "As we expected, there was no single approach performing well in every dataset."
