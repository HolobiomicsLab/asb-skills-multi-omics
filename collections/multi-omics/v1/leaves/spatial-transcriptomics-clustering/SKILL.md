---
name: spatial-transcriptomics-clustering
description: Use when analyzing spot-level ST data (Visium h5, ST-platform count matrices
  with H&E images) where the goal is to identify tissue domains or layers and you
  need to compare or choose among gene-only, spatially aware, and image-integration
  clustering strategies.
when_to_use_negative:
- When neither image features nor spatial structure are informative — the article
  advises relying solely on gene expression data with standard methods such as PCA
  instead of integration
- When samples are spatially uniform and the histological image is uninformative or
  conflicts with gene expression — skip modality integration and only employ a spatially
  aware process
- When analyzing data types outside the supported ST platforms (e.g., non-spot-based
  technologies or 3D spatial resolution, which the article only describes as future
  extensions)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: stImage
  role: Core R package providing LoadImageFeature, PreProcessing, FindSNNClusters,
    SpatialAwareProcess and MultiModalIntegration for the 54 integrative clustering
    strategies
  repo: https://github.com/YuWang-VUMC/stImage
- name: Seurat
  role: Underlying toolkit for dimensionality reduction, SNN clustering and spatial
    visualization (SCTransform, PCA, FindSNNClusters, SpatialDimPlot)
  repo: https://satijalab.org/seurat
- name: SpatialPCA
  role: Spatially aware dimensionality reduction run via SpatialAwareProcess on the
    'counts' slot (pcaDim_g=20, pcaDim_i=20) before WNN integration
- name: BayesSpace
  role: Bayesian spatial clustering run via SpatialAwareProcess on 'data' slot of
    SCT and ImageFeature assays
- name: tidyverse
  role: Data manipulation and result-table assembly (e.g., writing ARI comparison
    CSVs)
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_007
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:d65ad429ac3f8e0d5221e9de62cac922b24d9a9fe5f783c087f88771eb3ae8e0
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# spatial-transcriptomics-clustering

## Summary

Cluster spatial transcriptomics (ST) spots by integrating gene expression, histology image features, and spatial location using the stImage R framework, which offers 54 integrative strategies spanning single-modality, spatially aware (BayesSpace, SpatialPCA, stLearn), and multi-modal integration (WNN, MCIA, tICA, Spectrum) approaches.

## When to use

Use when analyzing spot-level ST data (Visium h5, ST-platform count matrices with H&E images) where the goal is to identify tissue domains or layers and you need to compare or choose among gene-only, spatially aware, and image-integration clustering strategies.

## When NOT to use

- When neither image features nor spatial structure are informative — the article advises relying solely on gene expression data with standard methods such as PCA instead of integration
- When samples are spatially uniform and the histological image is uninformative or conflicts with gene expression — skip modality integration and only employ a spatially aware process
- When analyzing data types outside the supported ST platforms (e.g., non-spot-based technologies or 3D spatial resolution, which the article only describes as future extensions)

## Inputs

- Spatial Transcriptomics gene count matrix (e.g., HER2 H1.tsv, GSE111672 PDAC-A counts)
- Spot coordinate table with pathologist/manual annotations (e.g., H1_labeled_coordinates.tsv, spatialLIBD layer labels)
- H&E histology image (e.g., HE_BT24044_D2.jpg)
- Visium filtered_feature_bc_matrix.h5 with spatial folder (mouse kidney, DLPFC 151673)
- Pre-computed VGG16 image-feature matrix and RGB quantile matrices (optional)

## Outputs

- Per-spot cluster assignments for each of the up to 54 strategies (gene-only, image-only, spatially aware, integrated)
- Spatial scatter plots of cluster assignments (SpatialDimPlot/AllSpatialDimPlot) and marker feature plots (SpatialFeaturePlot)
- ARI comparison table (e.g., HER2_H1_ARI_strategies.csv, dlpfc_151673_ari.csv)
- Diagnostic graphs reflecting spatial and histological characteristics to guide strategy choice

## How to apply

Build an stImage object from the count matrix, deep-learning image features (VGG16/ImageNet-derived via ExtractFeatures with patchSize 200) and RGB quantiles using LoadImageFeature, then run PreProcessing with SCT normalization and PCA (e.g., pcaDim_s=20, pcaDim_i=20, pcaDim_c=6, genePercentCut=0.25, imagePercentCut=0.2). Cluster each modality with FindSNNClusters on SCTPCA, ImageFeaturePCA and RGBPCA reductions, optionally apply SpatialAwareProcess (BayesSpace on 'data' slot, SpatialPCA on 'counts' with pcaDim_g=20, stLearn with cosine distance and weights_matrix_all), and combine modalities via MultiModalIntegration with WNN/MCIA/tICA/Spectrum. Choose the scenario per the article's guidance: spatial-aware process followed by modality integration when image features align with expression, direct modality integration (no spatial step) when spatial heterogeneity is prominent, spatially aware only when the image is uninformative, or PCA on gene expression alone when neither is informative. Validate by computing the adjusted Rand index against pathologist or manual annotations (e.g., HER2 H1 benchmark: SpatialPCA+WNN ARI 0.62 > WNN 0.58; DLPFC 151673: image-integrated WNN lowers ARI).

## Related tools

- **stImage** (Core R package providing LoadImageFeature, PreProcessing, FindSNNClusters, SpatialAwareProcess and MultiModalIntegration for the 54 integrative clustering strategies) — https://github.com/YuWang-VUMC/stImage
- **Seurat** (Underlying toolkit for dimensionality reduction, SNN clustering and spatial visualization (SCTransform, PCA, FindSNNClusters, SpatialDimPlot)) — https://satijalab.org/seurat
- **SpatialPCA** (Spatially aware dimensionality reduction run via SpatialAwareProcess on the 'counts' slot (pcaDim_g=20, pcaDim_i=20) before WNN integration)
- **BayesSpace** (Bayesian spatial clustering run via SpatialAwareProcess on 'data' slot of SCT and ImageFeature assays)
- **tidyverse** (Data manipulation and result-table assembly (e.g., writing ARI comparison CSVs))

## Examples

```
stImage::MultiModalIntegration(MultimodalMethod='WNN', reduction.list=list('SCTPCA','ImageFeaturePCA'), pcaDim_s=20, pcaDim_i=20, nCluster=6)
```

## Evaluation signals

- ARI of each strategy's cluster assignment against pathologist/manual annotations must reproduce benchmark orderings: HER2 H1 Gene/Image ~0.43-0.44 < SpatialPCA/stLearn ~0.45-0.48 < WNN 0.58 < SpatialPCA+WNN 0.62 (BayesSpace 0.29); DLPFC 151673 GeneBayesSpace ~0.56 > GeneSpatialPCA ~0.43 > gene-only ~0.36 > image-only ~0.18
- Spatial scatter of clusters must co-localize with marker signal (e.g., WNN glomerulus cluster overlapping Podxl/Synpo/Nphs1/Ehd3; arterial clusters with Mgp/Flna/Tagln/Eln) as in the paper's Figure 6
- Reproduced PDAC-A SpatialPCA+WNN map must show clusters 4/5/7 in cancer/desmoplasia, cluster 7 = clone A, cluster 5 = clone B, cluster 2 = ductal epithelium, cluster 0 = stroma, matching Fig. 4
- Preprocessing must drop the 21 ring pattern genes for the HER2 dataset and use the specified percent cuts (genePercentCut=0.25, imagePercentCut=0.2) and PC dimensions
- Result CSV (e.g., HER2_H1_ARI_strategies.csv) contains one ARI per strategy with the expected number of clusters (nCluster=6 for HER2 H1)

## Limitations

- No single approach performed well in every dataset — strategy effectiveness is highly dependent on transcriptomic, spatial, and image patterns, and image integration can worsen layer identification (DLPFC WNN variants lower ARI)
- Multi-modal integration is computationally more intensive and algorithm-dependent, though stImage runtime remains manageable on a typical office desktop
- WNN integration does not use spatial information; stImage is extensible to additional modalities and 3D spatial resolution only as a future direction
- Supplementary Fig. S13/Table S1 (four scenarios guiding strategy choice) content is not provided in the available context

## Evidence

- [abstract] By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information.: "By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information."
- [discussion] we recommend applying a spatial aware process followed by modality integration to fully leverage the complementary information: "we recommend applying a spatial aware process followed by modality integration to fully leverage the complementary information"
- [discussion] skipping the modality integration and only employing a spatially aware process can be advantageous: "skipping the modality integration and only employing a spatially aware process can be advantageous"
- [methods] remove the 21 ring pattern genes mentioned in the orginal paper: "remove the 21 ring pattern genes mentioned in the orginal paper"
- [readme] StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images.: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images."
