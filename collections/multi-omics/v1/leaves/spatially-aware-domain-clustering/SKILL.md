---
name: spatially-aware-domain-clustering
description: Use when spot-level ST data (counts, spot coordinates, H&E image) must
  be partitioned into spatial tissue domains whose labels should align with histologically
  annotated regions such as tumor clones, stroma, or epithelium.
when_to_use_negative:
- Samples are spatially uniform and the histological image is uninformative or conflicts
  with gene expression — the article advises skipping modality integration and only
  employing a spatially aware process
- Spatial heterogeneity in tissue structures is prominent — skip the spatial aware
  process and proceed directly to modality integration
- Neither image features nor spatial structure are informative — rely solely on gene
  expression with standard methods such as PCA
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: stImage
  role: Framework providing LoadImageFeature, PreProcessing, FindSNNClusters, SpatialAwareProcess,
    and MultiModalIntegration for spatially aware domain clustering
  repo: https://github.com/YuWang-VUMC/stImage
- name: SpatialPCA
  role: Spatially aware dimensionality-reduction method invoked via SpatialAwareMethod='SpatialPCA'
    to model spatial correlation across tissue locations
  repo: https://github.com/shangll123/SpatialPCA
- name: Seurat
  role: Underlying R object/normalization infrastructure (SCT normalization, PCA,
    SNN clustering) used by stImage workflows
  repo: https://satijalab.org/seurat
- name: tidyverse
  role: R data-manipulation and plotting utilities used in the stImage workflow scripts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:84879430a20f403ec9a34f78da4fd082dcb412e289e438e71d22a9ed30343950
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
yes_rate: 0.5625
n_claims: 16
---

# spatially-aware-domain-clustering

## Summary

Clusters spatial transcriptomics spots into tissue domains by running a spatially aware dimensionality reduction (e.g., SpatialPCA via stImage) on gene-expression and deep-histology-feature assays, then integrating the two modalities (e.g., WNN) so domain labels respect both transcriptomic and image structure.

## When to use

Use when spot-level ST data (counts, spot coordinates, H&E image) must be partitioned into spatial tissue domains whose labels should align with histologically annotated regions such as tumor clones, stroma, or epithelium.

## When NOT to use

- Samples are spatially uniform and the histological image is uninformative or conflicts with gene expression — the article advises skipping modality integration and only employing a spatially aware process
- Spatial heterogeneity in tissue structures is prominent — skip the spatial aware process and proceed directly to modality integration
- Neither image features nor spatial structure are informative — rely solely on gene expression with standard methods such as PCA

## Inputs

- Spatial transcriptomics gene-expression count matrix (e.g., ST platform data such as PDAC-A from GEO GSE111672)
- Spot/tissue coordinates of the ST array
- H&E histology image paired to the ST spots
- VGG16-derived image features and RGB quantiles per spot (via LoadImageFeature)

## Outputs

- Per-spot domain cluster labels (gene-only SpatialPCA, image-only SpatialPCA, and SpatialPCA+WNN)
- Spatial scatter plots of clusters over the tissue (AllSpatialDimPlot)
- Integrated WNN reduction object (e.g., 'SCTSpatialPCA' + 'ImageFeatureSpatialPCA' merged)

## How to apply

Load the ST data with stImage's LoadImageFeature, pairing gene counts with VGG16-derived image features and RGB quantiles. Run PreProcessing with SCT normalization and PCA on both the Spatial and ImageFeature assays (e.g., 20 PCs per modality, genePercentCut 0.25, imagePercentCut 0.2), then obtain initial SNN clusters with FindSNNClusters. Run SpatialAwareProcess with SpatialAwareMethod='SpatialPCA' separately on the SCT gene assay (slot 'counts', pcaDim_g=20, percentCut=0.25) and the ImageFeature assay (pcaDim_i=20, percentCut=0.2). Integrate the two SpatialPCA reductions with MultiModalIntegration (MultimodalMethod='WNN', reduction.list=list('SCTSpatialPCA','ImageFeatureSpatialPCA')) to get joint WNN clusters. Validate by plotting clusters spatially (AllSpatialDimPlot) and checking that domain labels reproduce the expected histology layout (e.g., in PDAC-A, clusters 4/5/7 confined to cancer/desmoplasia with clone A in cluster 7 and clone B in cluster 5).

## Related tools

- **stImage** (Framework providing LoadImageFeature, PreProcessing, FindSNNClusters, SpatialAwareProcess, and MultiModalIntegration for spatially aware domain clustering) — https://github.com/YuWang-VUMC/stImage
- **SpatialPCA** (Spatially aware dimensionality-reduction method invoked via SpatialAwareMethod='SpatialPCA' to model spatial correlation across tissue locations) — https://github.com/shangll123/SpatialPCA
- **Seurat** (Underlying R object/normalization infrastructure (SCT normalization, PCA, SNN clustering) used by stImage workflows) — https://satijalab.org/seurat
- **tidyverse** (R data-manipulation and plotting utilities used in the stImage workflow scripts)

## Examples

```
MultiModalIntegration(obj = st_obj, reduction.list = list('SCTSpatialPCA', 'ImageFeatureSpatialPCA'), MultimodalMethod = 'WNN')
```

## Evaluation signals

- Reproduced SpatialPCA+WNN spatial map reproduces the published domain layout (e.g., PDAC-A Fig. 4: cancer/desmoplasia clusters 4/5/7, ductal epithelium cluster 2, stroma cluster 0 in the same tissue regions)
- Cancer-associated clusters fall exclusively within the cancer cells and desmoplasia region, with clone A distinguished in cluster 7 from clone B in cluster 5
- Both modality-specific SpatialPCA runs complete with the specified PCA dimensions (20 PCs) and percentCut thresholds (0.25 gene, 0.2 image) before WNN integration
- WNN integration succeeds with reduction.list = list('SCTSpatialPCA','ImageFeatureSpatialPCA') and produces per-spot cluster labels exportable from AllSpatialDimPlot
- Diagnostic-graph guidance agrees that image features align well with gene expression for the given dataset before choosing spatial-aware-process + modality integration

## Limitations

- No single integration approach performed well on every dataset tested; the article recommends using its diagnostic graphs to pick a strategy
- Multi-modal integration is computationally more intensive and algorithm-dependent, though stImage runtime remains manageable on a typical office desktop
- Feature extraction requires a functioning tensorflow/keras R installation (python virtualenv 'r-reticulate'), a common installation failure point per the README

## Evidence

- [intro] SpatialPCA [11] models the spatial correlation structure across tissue locations during dimensionality reduction: "SpatialPCA [11] models the spatial correlation structure across tissue locations during dimensionality reduction"
- [abstract] By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information.: "By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information."
- [discussion] we recommend applying a spatial aware process followed by modality integration to fully leverage the complementary information: "we recommend applying a spatial aware process followed by modality integration to fully leverage the complementary information"
- [readme] StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images.: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images."
