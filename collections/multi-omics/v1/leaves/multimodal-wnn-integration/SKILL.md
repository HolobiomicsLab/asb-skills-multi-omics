---
name: multimodal-wnn-integration
description: Use when spot-level ST data have been normalized (SCT) with image features
  extracted (VGG16/RGB via LoadImageFeature) and you need spot clustering that jointly
  weighs gene expression and histology modality after a spatially aware process.
when_to_use_negative:
- The histological image is uninformative or conflicts with gene expression — the
  article advises skipping modality integration and employing only a spatially aware
  process
- Neither image features nor spatial structure are informative — rely solely on gene
  expression data with standard methods such as PCA
- Data are not spot-resolution spatial transcriptomics with aligned histology images
  (e.g. already-aggregated feature tables without spatial coordinates)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: stImage
  role: Provides SpatialAwareProcess, MultiModalIntegration (WNN), FindSNNClusters
    and AllSpatialDimPlot used to fuse and cluster the modalities
  repo: https://github.com/YuWang-VUMC/stImage
- name: SpatialPCA
  role: Spatially aware dimensionality reduction applied to each modality assay before
    WNN fusion
  repo: https://github.com/shangll123/SpatialPCA
- name: Seurat
  role: Underlying R toolkit for SNN clustering and Seurat-object handling in the
    stImage workflow
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:add8b681437018bbfa36035e4077ada1cad3fd17bf6a5a3c356be4683c1d59f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuWang-VUMC/stImage
  tools:
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
yes_rate: 0.5833333333333334
n_claims: 12
---

# multimodal-wnn-integration

## Summary

Integrates gene-expression-derived and histology-image-derived SpatialPCA reductions with weighted nearest neighbor (WNN) fusion via stImage's MultiModalIntegration, producing spatial cluster labels that reflect both transcriptomic and image modalities.

## When to use

Use when spot-level ST data have been normalized (SCT) with image features extracted (VGG16/RGB via LoadImageFeature) and you need spot clustering that jointly weighs gene expression and histology modality after a spatially aware process.

## When NOT to use

- The histological image is uninformative or conflicts with gene expression — the article advises skipping modality integration and employing only a spatially aware process
- Neither image features nor spatial structure are informative — rely solely on gene expression data with standard methods such as PCA
- Data are not spot-resolution spatial transcriptomics with aligned histology images (e.g. already-aggregated feature tables without spatial coordinates)

## Inputs

- stImage object with SCT-normalized Spatial assay and ImageFeature assay (VGG16 features + RGB quantiles from LoadImageFeature)
- Spot spatial coordinates from the ST platform (e.g. PDAC-A ST dataset, GEO GSE111672)
- Per-modality PCA/SpatialPCA reduction objects (SCTSpatialPCA, ImageFeatureSpatialPCA)

## Outputs

- WNN-integrated dimensionality reduction (e.g. 'wnn' reduction) on the stImage object
- Per-spot WNN cluster labels (FindSNNClusters output)
- Spatial scatter plots of gene-only, image-only, and WNN clusters over the tissue (AllSpatialDimPlot)

## How to apply

First run SpatialAwareProcess with SpatialAwareMethod='SpatialPCA' separately on the SCT gene assay (slot 'counts', pcaDim_g=20, percentCut=0.25) and on the ImageFeature assay (pcaDim_i=20, percentCut=0.2). Then call MultiModalIntegration on the two SpatialPCA outputs with reduction.list=list('SCTSpatialPCA','ImageFeatureSpatialPCA') and MultimodalMethod='WNN', so each spot is weighted by modality-specific quality rather than averaged. Obtain final spot labels with FindSNNClusters and visualize the gene-only, image-only, and WNN maps side by side with AllSpatialDimPlot to confirm the fused clusters occupy coherent tissue regions. Rationale: stImage offers 54 integrative strategies and its diagnostic graphs indicate that spatially aware processing followed by modality integration is appropriate when image features align well with gene expression.

## Related tools

- **stImage** (Provides SpatialAwareProcess, MultiModalIntegration (WNN), FindSNNClusters and AllSpatialDimPlot used to fuse and cluster the modalities) — https://github.com/YuWang-VUMC/stImage
- **SpatialPCA** (Spatially aware dimensionality reduction applied to each modality assay before WNN fusion) — https://github.com/shangll123/SpatialPCA
- **Seurat** (Underlying R toolkit for SNN clustering and Seurat-object handling in the stImage workflow) — https://github.com/satijalab/seurat

## Examples

```
R: st <- MultiModalIntegration(st, reduction.list = list('SCTSpatialPCA', 'ImageFeatureSpatialPCA'), MultimodalMethod = 'WNN')
```

## Evaluation signals

- The WNN spatial map reproduces the published domain layout — on PDAC-A, clusters 4, 5, and 7 fall exclusively within the cancer cells and desmoplasia region, cluster 2 marks ductal epithelium, and cluster 0 marks stroma
- Both modality-specific SpatialPCA runs complete with the intended PC counts (pcaDim_g=20 gene, pcaDim_i=20 image) before integration
- WNN clusters differ from (and are more coherent than) either gene-only or image-only SpatialPCA clustering, indicating the fusion actually weights both modalities
- Per-spot cluster labels export cleanly and can be overlaid on the H&E image via AllSpatialDimPlot without spot misalignment

## Limitations

- No single integration approach performed well on every dataset tested; strategy choice should be guided by stImage's diagnostic graph
- Multi-modal integration is computationally more intensive and algorithm-dependent, though runtime remains manageable on a typical office desktop
- When spatial heterogeneity in tissue structures is prominent, the article recommends skipping the spatial aware process and proceeding directly to modality integration, i.e. the SpatialPCA+WNN ordering may not fit every sample

## Evidence

- [abstract] By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information.: "By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information."
- [discussion] we recommend applying a spatial aware process followed by modality integration to fully leverage the complementary information: "we recommend applying a spatial aware process followed by modality integration to fully leverage the complementary information"
- [readme] StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images.: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images."
- [intro] SpatialPCA [11] models the spatial correlation structure across tissue locations during dimensionality reduction: "SpatialPCA [11] models the spatial correlation structure across tissue locations during dimensionality reduction"
