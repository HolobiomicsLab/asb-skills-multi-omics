---
name: clustering-ari-benchmark-evaluation
description: Use when a spatial transcriptomics sample has manual/domain annotations
  (e.g., DLPFC layer labels) and multiple candidate clusterings from different pipelines
  must be ranked to decide whether image integration helps or hurts.
when_to_use_negative:
- The sample has no manual layer annotations or other ground-truth labels, so adjusted
  Rand index cannot be computed against a reference
- The goal is exploratory clustering of a new dataset rather than benchmarking/choosing
  an analysis strategy — stImage's diagnostic graphs (Distance Consistency, Spot Distance
  To Neighbors, Similarity To Neighbors) are designed for that decision instead
- Only one clustering result exists and there is nothing to compare, making a ranking
  table meaningless
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: stImage
  role: Framework providing LoadImageFeature, PreProcessing, FindSNNClusters, SpatialAwareProcess
    and MultiModalIntegration to generate the clusterings being benchmarked
  repo: https://github.com/YuWang-VUMC/stImage
- name: Seurat
  role: Underlying toolkit for SNN clustering (FindSNNClusters) and PCA reductions
    used by stImage
  repo: https://satijalab.org/seurat
- name: SpatialPCA
  role: Spatially aware dimensionality-reduction clustering variant whose ARI is compared
    against its gene-only counterpart
  repo: https://github.com/shangll123/SpatialPCA
- name: BayesSpace
  role: Bayesian spatial clustering variant benchmarked as GeneBayesSpace and its
    WNN/Spectrum image-integrated forms
  repo: https://bioconductor.org/packages/BayesSpace
- name: tidyverse
  role: Used in the stImage methods for assembling and writing the ARI comparison
    table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:3ea94120133c25d5d7d839c051a13005ae2c3cd11a1ab71f92a49b56dd863904
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# clustering-ari-benchmark-evaluation

## Summary

Benchmark spatial transcriptomics clustering strategies (gene-only, spatially aware, and histology-image-integrated) by computing adjusted Rand index (ARI) of each clustering against manual layer annotations, as done in stImage on the human DLPFC Visium sample 151673.

## When to use

Use when a spatial transcriptomics sample has manual/domain annotations (e.g., DLPFC layer labels) and multiple candidate clusterings from different pipelines must be ranked to decide whether image integration helps or hurts.

## When NOT to use

- The sample has no manual layer annotations or other ground-truth labels, so adjusted Rand index cannot be computed against a reference
- The goal is exploratory clustering of a new dataset rather than benchmarking/choosing an analysis strategy — stImage's diagnostic graphs (Distance Consistency, Spot Distance To Neighbors, Similarity To Neighbors) are designed for that decision instead
- Only one clustering result exists and there is nothing to compare, making a ranking table meaningless

## Inputs

- Gene expression count matrix with spot coordinates (Visium DLPFC sample 151673 from spatialLIBD)
- H&E histology image with deep-learning features (VGG16 ImageNet, patchSize 200) and RGB quantiles
- Manual layer annotations per spot (ground-truth labels)
- Clustering assignments from each pipeline (SCTPCA, ImageFeaturePCA, SpatialPCA, BayesSpace, stLearn, WNN/MCIA/tICA/Spectrum integrations)

## Outputs

- ARI comparison table (dlpfc_151673_ari.csv) ranking all clustering strategies
- Per-strategy ARI scores against manual layer annotations

## How to apply

Run each competing clustering pipeline on the same sample: gene-only SCT-PCA clustering, spatially aware variants (SpatialPCA on 'counts', BayesSpace on 'data' with platform='ST'), image-feature clustering (ImageFeaturePCA from VGG16 patches, patchSize 200), stLearn image-similarity refinement, and multi-modal integrations (WNN, MCIA, tICA, Spectrum) built on the SCTPCA/ImageFeaturePCA reductions. Cluster each strategy with FindSNNClusters at the same target cluster number matching the number of annotated layers. Compute the adjusted Rand index of every clustering against the manual layer annotations (sample 151673) and write the ARI comparison table to dlpfc_151673_ari.csv. Interpret results comparatively, not absolutely: the stImage paper found no single approach performing well on every dataset, so the ARI ordering — e.g., GeneBayesSpace ~0.56 > GeneSpatialPCA ~0.43 > gene-only ~0.36 > image-only ~0.18, with WNN image integration lowering ARI and stLearn slightly improving it — is the decision signal for strategy selection.

## Related tools

- **stImage** (Framework providing LoadImageFeature, PreProcessing, FindSNNClusters, SpatialAwareProcess and MultiModalIntegration to generate the clusterings being benchmarked) — https://github.com/YuWang-VUMC/stImage
- **Seurat** (Underlying toolkit for SNN clustering (FindSNNClusters) and PCA reductions used by stImage) — https://satijalab.org/seurat
- **SpatialPCA** (Spatially aware dimensionality-reduction clustering variant whose ARI is compared against its gene-only counterpart) — https://github.com/shangll123/SpatialPCA
- **BayesSpace** (Bayesian spatial clustering variant benchmarked as GeneBayesSpace and its WNN/Spectrum image-integrated forms) — https://bioconductor.org/packages/BayesSpace
- **tidyverse** (Used in the stImage methods for assembling and writing the ARI comparison table)

## Examples

```
ari <- mclust::adjustedRandIndex(clusters, manual_layers); write.csv(ari_table, "dlpfc_151673_ari.csv")
```

## Evaluation signals

- Reproduce the benchmark ordering: GeneBayesSpace ~0.56 > GeneSpatialPCA ~0.43 > gene-only ~0.36 > image-only ~0.18 on DLPFC sample 151673
- Every image-integrated WNN variant (WNN, SpatialPCA+WNN, BayesSpace+WNN) must show lower ARI than its gene-only counterpart
- stLearn (cosine distance, weights_matrix_all, percentCut 0.03) must show slightly higher ARI than the gene-only baseline
- All ARIs fall in [0, 1] and every clustering used the same target cluster number matching the manual annotation layer count
- Output file dlpfc_151673_ari.csv contains one ARI row per evaluated strategy

## Limitations

- No single integration approach performed well on every dataset — the DLPFC ordering does not transfer blindly to other tissues or platforms
- Image-integrated modalities (WNN, MCIA, tICA, Spectrum) are computationally more intensive and algorithm-dependent
- Results are only as meaningful as the manual annotations; ARI penalizes disagreement even when alternative layer definitions are biologically defensible

## Evidence

- [abstract] 54 integrative strategies spanning gene-only, spatial-aware, and image-integrated clusterings: "By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information."
- [discussion] No universal winner across datasets: "As we expected, there was no single approach performing well in every dataset."
- [intro] stImage enables simultaneous comparison of methods for data-driven decisions: "stImage enables users to apply and compare different methods simultaneously, as well as make data-driven decisions using its diagnostic graphs that reflect the spatial and histological"
- [readme] stImage installation and R environment from README: "Once Tensorflow and keras were successfully installed, you can then install the latest version of stImage from GitHub with:

```r
library(devtools)
install_github("YuWang-VUMC/stImage")
```"
