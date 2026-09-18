---
name: histology-image-feature-extraction
description: Use when raw H&E images paired with ST spot coordinates or Visium (filtered_feature_bc_matrix.h5
  + spatial folder) must be turned into per-spot VGG16 image-feature and RGB quantile
  matrices as the image modality of an stImage object.
when_to_use_negative:
- When a pre-computed image-feature matrix already exists for the dataset (the DLPFC
  workflow explicitly allows loading one instead of re-extracting).
- When the histological image is uninformative or conflicts with gene expression —
  the article recommends skipping modality integration and using only a spatially
  aware process or gene expression alone.
- When the tissue sample is spatially uniform; spatial-aware-only or gene-only PCA
  strategies are preferred over image-feature-based integration.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: stImage
  role: Provides ExtractFeatures/ExtractFeaturesVisium (VGG16-derived ImageNet features
    and RGB quantiles) and LoadImageFeature/LoadImageFeatureVisium to assemble the
    image modality into the analysis object
  repo: https://github.com/YuWang-VUMC/stImage
- name: Seurat
  role: Downstream framework on which stImage builds the ImageFeature assay, ImageFeaturePCA
    reduction, and clustering of the image modality
  repo: https://satijalab.org/seurat
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:f75329cf7878f778a4b71ab47142565246cde8ab5908d3fe75c68f6f017c44b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuWang-VUMC/stImage
  tools:
  - BayesSpace
  - R
  - Seurat
  - SpatialPCA
  - patchwork
  - stImage
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.38461538461538464
n_claims: 13
---

# histology-image-feature-extraction

## Summary

Converts H&E histology images accompanying spatial transcriptomics (ST/Visium) data into per-spot deep-learning image features and RGB quantile matrices using stImage's VGG16-based ExtractFeatures, so the image modality can be integrated with gene expression.

## When to use

Use when raw H&E images paired with ST spot coordinates or Visium (filtered_feature_bc_matrix.h5 + spatial folder) must be turned into per-spot VGG16 image-feature and RGB quantile matrices as the image modality of an stImage object.

## When NOT to use

- When a pre-computed image-feature matrix already exists for the dataset (the DLPFC workflow explicitly allows loading one instead of re-extracting).
- When the histological image is uninformative or conflicts with gene expression — the article recommends skipping modality integration and using only a spatially aware process or gene expression alone.
- When the tissue sample is spatially uniform; spatial-aware-only or gene-only PCA strategies are preferred over image-feature-based integration.

## Inputs

- H&E histology image (e.g. HE_BT24044_D2.jpg) paired with ST spot coordinates
- Visium spaceranger outputs: filtered_feature_bc_matrix.h5 and spatial folder with raw H&E image
- Spot position table with spot names formatted as X<x>x<y>
- Patch size parameter (200 px) and feature-model choice (VGG16, ImageNet weights)

## Outputs

- Per-spot deep-learning image-feature matrix (rows = spots, columns = VGG16 features)
- Per-spot RGB quantile matrix
- stImage object built with LoadImageFeature/LoadImageFeatureVisium containing the ImageFeature assay
- ImageFeaturePCA reduction (e.g. dims 1:20–1:30) available for clustering and MultiModalIntegration

## How to apply

First install the stImage dependencies, ensuring tensorflow and keras are working in an R reticulate environment ('r-reticulate') as the README instructs, since feature extraction uses the VGG16 (ImageNet) deep-learning model. For ST-platform data (e.g. HER2 breast cancer H1 from Zenodo 10.5281/zenodo.4751624), tile the H&E image (e.g. HE_BT24044_D2.jpg) around each spot, formatting spot names as X<x>x<y> and setting patch size to 200, to obtain the image-feature matrix and RGB quantile matrix. For Visium data (e.g. mouse kidney or DLPFC 151673), use stImage::ExtractFeaturesVisium on the raw H&E image with the same VGG16 ImageNet model and patchSize 200, or load a pre-computed feature matrix if one exists. Load both matrices into the object with LoadImageFeature (or LoadImageFeatureVisium), pairing them with the count table and position table. Downstream, the image modality is reduced with ImageFeaturePCA (pcaDim_i=20–30) inside stImage::PreProcessing (imagePercentCut=0.2, prefilterimagePercentCut=0.1), which feeds integration strategies such as WNN or SpatialPCA+WNN.

## Related tools

- **stImage** (Provides ExtractFeatures/ExtractFeaturesVisium (VGG16-derived ImageNet features and RGB quantiles) and LoadImageFeature/LoadImageFeatureVisium to assemble the image modality into the analysis object) — https://github.com/YuWang-VUMC/stImage
- **Seurat** (Downstream framework on which stImage builds the ImageFeature assay, ImageFeaturePCA reduction, and clustering of the image modality) — https://satijalab.org/seurat

## Examples

```
library(stImage); features <- ExtractFeatures(image = "HE_BT24044_D2.jpg", positionTable = pos, patchSize = 200, model = "VGG16")
```

## Evaluation signals

- The extracted feature matrix and RGB quantile matrix have one row per tissue spot, with spot names matching the X<x>x<y> format of the position table (e.g. H1_labeled_coordinates.tsv).
- LoadImageFeature/LoadImageFeatureVisium succeeds in building an stImage object whose ImageFeature assay pairs 1:1 with the gene count table.
- PreProcessing on the ImageFeature assay runs with imagePercentCut=0.2 and prefilterimagePercentCut=0.1 and yields an ImageFeaturePCA reduction with the requested dimensionality (pcaDim_i=20 for ST/HER2 and DLPFC, 30 for mouse kidney Visium).
- tensorflow is importable in the R session (tf$constant("Hello Tensorflow!") runs) confirming the VGG16/keras dependency stack is functional.
- Downstream WNN clustering on SCTPCA + ImageFeaturePCA reproduces expected biological signals (e.g. glomerulus cluster co-localized with Podxl/Synpo/Nphs1/Ehd3 in mouse kidney).

## Limitations

- Requires a working tensorflow/keras Python environment in R (reticulate 'r-reticulate' virtualenv); extraction fails if installation instructions are not followed.
- No single integration approach performed well on every dataset tested; image features can hurt performance (e.g. in DLPFC, WNN integration lowered ARI versus gene-only).
- Feature informativeness depends on image quality; the article notes assumptions about image quality/informativeness are not always met.

## Evidence

- [abstract] By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information.: "By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information."
- [readme] stImage is freely available at https://github.com/YuWang-VUMC/stImage.: "StImage is freely available at https://github.com/YuWang-VUMC/stImage."
- [readme] Users should also make sure to successfully install the *tensorflow* and *keras* following instruction Tensorflow for R.: [evidence span withheld — anchor status: external_doc]
- [intro] stImage enables users to apply and compare different methods simultaneously, as well as make data-driven decisions using its diagnostic graphs.: "stImage enables users to apply and compare different methods simultaneously, as well as make data-driven decisions using its diagnostic graphs that reflect the spatial and histological"
- [discussion] skipping the modality integration and only employing a spatially aware process can be advantageous: "skipping the modality integration and only employing a spatially aware process can be advantageous"
