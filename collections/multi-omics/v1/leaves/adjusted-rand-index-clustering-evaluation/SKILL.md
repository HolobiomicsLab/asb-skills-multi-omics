---
name: adjusted-rand-index-clustering-evaluation
description: Use when spatial transcriptomics cluster label sets from multiple integrative
  strategies must be scored against known ground-truth labels (e.g.
when_to_use_negative:
- The dataset has no ground-truth (simulated or manually annotated) cluster labels
  — ARI against an unknown truth is undefined and cannot rank strategies
- Comparing probabilistic/soft cluster memberships rather than hard spot-level label
  partitions of the same spots
- The goal is strategy selection on real data without labels — the article instead
  recommends stImage's diagnostic graphs for data-driven strategy choice
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: stImage
  role: R package whose LoadImageFeature, PreProcessing, FindSNNClusters, SpatialAwareProcess,
    and MultiModalIntegration functions produce the cluster label sets being ARI-scored
  repo: https://github.com/YuWang-VUMC/stImage
- name: Seurat
  role: Provides the single-cell object framework and PCA reductions (SCTPCA, ImageFeaturePCA)
    that feed clustering and integration
  repo: https://satijalab.org/seurat
- name: BayesSpace
  role: Spatially aware clustering engine invoked via SpatialAwareProcess with platform
    = 'ST'
- name: SpatialPCA
  role: Spatial dimensionality reduction used in SpatialAwareProcess with pcaDim_g
    = pcaDim_i = 20
- name: tidyverse
  role: R data-wrangling used to assemble the ARI results table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:c0159988754a6ed16d87864de81c5e520e46b642f5137dba1c27bd096658c2e2
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2
n_claims: 10
---

# adjusted-rand-index-clustering-evaluation

## Summary

Benchmark protocol for ranking stImage spatial transcriptomics integrative strategies (single-modality, SpatialAwareProcess, MultiModalIntegration) by computing the Adjusted Rand Index (ARI) of each strategy's cluster labels against simulated ground-truth cluster labels.

## When to use

Use when spatial transcriptomics cluster label sets from multiple integrative strategies must be scored against known ground-truth labels (e.g. simulated ST data) to determine which strategy — gene-only, image-only, spatially aware, or modality integration — achieves the best clustering agreement.

## When NOT to use

- The dataset has no ground-truth (simulated or manually annotated) cluster labels — ARI against an unknown truth is undefined and cannot rank strategies
- Comparing probabilistic/soft cluster memberships rather than hard spot-level label partitions of the same spots
- The goal is strategy selection on real data without labels — the article instead recommends stImage's diagnostic graphs for data-driven strategy choice

## Inputs

- Simulated spatial transcriptomics gene expression matrix with ground-truth cluster labels
- Image feature matrix (deep learning-derived histology features)
- RGB quantile matrix
- Cluster label vectors from each strategy (FindSNNClusters SCTPCA/ImageFeaturePCA, SpatialAwareProcess, MultiModalIntegration WNN/MCIA/tICA/Spectrum)

## Outputs

- ARI score per integrative strategy
- Benchmark results table (ari_benchmark_results.csv)
- Ranked ordering of strategies: integration > single-modality, WNN and Spectrum highest

## How to apply

Build the analysis object with stImage::LoadImageFeature from the simulated ST data (gene expression matrix, image feature matrix, RGB quantile matrix, and ground-truth labels), then run stImage::PreProcessing (SCT normalization, PCA with 20 dims for the gene and image assays, pcaDim_c = 6) and obtain single-modality baselines with stImage::FindSNNClusters on the SCTPCA and ImageFeaturePCA reductions at the simulated cluster number. Run stImage::SpatialAwareProcess per modality using BayesSpace (platform = 'ST') and SpatialPCA (pcaDim_g = pcaDim_i = 20), and stImage::MultiModalIntegration with MultimodalMethod = c('WNN', 'MCIA', 'tICA', 'Spectrum') over the SCT and ImageFeature PCA reductions (genePercentCut = 0.25, imagePercentCut = 0.2, pcaDim_s = pcaDim_i = 20), plus combined spatially aware graphs with WNN and Spectrum. Compute the Adjusted Rand Index of every strategy's labels against the ground truth, assemble the table, and write ari_benchmark_results.csv. The rationale: with simulated labels known, ARI gives a partition-comparison metric invariant to label permutation, allowing fair ranking of the 54 integrative strategies stImage offers.

## Related tools

- **stImage** (R package whose LoadImageFeature, PreProcessing, FindSNNClusters, SpatialAwareProcess, and MultiModalIntegration functions produce the cluster label sets being ARI-scored) — https://github.com/YuWang-VUMC/stImage
- **Seurat** (Provides the single-cell object framework and PCA reductions (SCTPCA, ImageFeaturePCA) that feed clustering and integration) — https://satijalab.org/seurat
- **BayesSpace** (Spatially aware clustering engine invoked via SpatialAwareProcess with platform = 'ST')
- **SpatialPCA** (Spatial dimensionality reduction used in SpatialAwareProcess with pcaDim_g = pcaDim_i = 20)
- **tidyverse** (R data-wrangling used to assemble the ARI results table)

## Examples

```
write.csv(ari_benchmark_results, file = "ari_benchmark_results.csv")
```

## Evaluation signals

- ARI values are bounded in [-1, 1]; a self-comparison of ground truth against itself must yield exactly 1.0
- Under high dropout rates, WNN and Spectrum modality integration must achieve significantly higher ARI than MCIA and tICA, which in turn outperform gene-only and image-only baselines
- All ARI computations use the same simulated spot set and the same ground-truth label vector, so rankings are comparable across strategies
- ari_benchmark_results.csv is written and contains one ARI per strategy (single-modality, spatially aware, and integrated)
- Expected reported ordering holds: combined gene+image integration strategies outperform single-modality strategies

## Limitations

- ARI requires ground-truth labels, so this benchmark protocol applies to simulated data or manually annotated sections, not unsupervised real-data evaluation
- No single stImage integration approach performed well on every dataset — ARI rankings on one simulated dataset do not generalize; use diagnostic graphs to guide strategy choice per dataset
- Multi-modal integration is computationally more intensive and algorithm-dependent, though stImage runtime remains manageable on a typical office desktop
- MultiModalIntegration parameters (genePercentCut = 0.25, imagePercentCut = 0.2, pcaDim_s = pcaDim_i = 20) are dataset- and simulation-specific and may need retuning elsewhere

## Evidence

- [methods] Compute the Adjusted Rand Index of every strategy's cluster labels against the simulated ground-truth labels and assemble the results into a table.: [evidence span withheld — anchor status: not_found]
- [results] On simulated ST data, integrating gene and image modalities outperformed single-modality strategies: [evidence span withheld — anchor status: not_found]
- [discussion] As we expected, there was no single approach performing well in every dataset.: "As we expected, there was no single approach performing well in every dataset."
- [readme] StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images.: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images."
- [methods] Run stImage::MultiModalIntegration with MultimodalMethod = c('WNN', 'MCIA', 'tICA', 'Spectrum') over the SCT and ImageFeature PCA reductions: [evidence span withheld — anchor status: not_found]
