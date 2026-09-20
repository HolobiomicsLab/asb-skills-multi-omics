---
name: weighted-nearest-neighbor-multimodal-integration
description: Use when Visium spatial transcriptomics data has H&E image features (e.g.
  VGG16/ImageNet-derived) extracted and per-modality clusters miss fine substructures,
  and you need a non-spatial modality integration to identify components like glomeruli
  or arterial spots.
when_to_use_negative:
- When tissue is spatially uniform and the histological image is uninformative or
  conflicts with gene expression — the article recommends skipping modality integration
  and using a spatially aware process or gene-only PCA
- When spatial heterogeneity is prominent and spatially aware processing is preferable
  before integration (skip spatial-aware and integrate directly only per the diagnostic
  graph)
- When neither image features nor spatial structure are informative — rely solely
  on gene expression with standard methods such as PCA
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0622
tools:
- name: stImage
  role: R package providing ExtractFeaturesVisium, LoadImageFeatureVisium, PreProcessing,
    FindSNNClusters and MultiModalIntegration used to run the WNN fusion of SCTPCA
    and ImageFeaturePCA reductions
  repo: https://github.com/YuWang-VUMC/stImage
- name: Seurat
  role: Underlying framework for SCT normalization, PCA reductions and SNN clustering
    leveraged by stImage's MultiModalIntegration (WNN)
  repo: https://github.com/satijalab/seurat
- name: tidyverse
  role: R data-manipulation/plotting dependency used in the stImage workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:91eaaeefc9c96b9b785bcebbebbcae2a62e77df76ea1ac83790546aefaa88562
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuWang-VUMC/stImage
  tools:
  - R
  - Seurat
  - patchwork
  - stImage
  - tidyverse
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# weighted-nearest-neighbor-multimodal-integration

## Summary

Integration of transcriptomic SCT-PCA and deep histology image-feature PCA embeddings via weighted nearest neighbor (WNN) modality fusion in stImage/Seurat, used when spatially aware methods fail to recover fine substructures such as the glomerulus in mouse kidney Visium data.

## When to use

Use when Visium spatial transcriptomics data has H&E image features (e.g. VGG16/ImageNet-derived) extracted and per-modality clusters miss fine substructures, and you need a non-spatial modality integration to identify components like glomeruli or arterial spots.

## When NOT to use

- When tissue is spatially uniform and the histological image is uninformative or conflicts with gene expression — the article recommends skipping modality integration and using a spatially aware process or gene-only PCA
- When spatial heterogeneity is prominent and spatially aware processing is preferable before integration (skip spatial-aware and integrate directly only per the diagnostic graph)
- When neither image features nor spatial structure are informative — rely solely on gene expression with standard methods such as PCA

## Inputs

- 10x Visium filtered_feature_bc_matrix.h5 and spatial folder (mouse kidney Visium ST data)
- Raw H&E tissue image for VGG16-derived ImageNet feature and RGB quantile matrix extraction
- stImage object built by LoadImageFeatureVisium

## Outputs

- WNN-integrated spot cluster assignments stored as 'wnn_clusterN' in the stImage/Seurat object
- SpatialDimPlot panels of SCT, ImageFeature, and WNN cluster overlays on tissue space
- AllSpatialFeaturePlot/SpatialFeaturePlot (cluster.frame=TRUE) panels for Podxl, Synpo, Nphs1, Ehd3 (glomeruli) and Mgp, Flna, Tagln, Eln (arteries)

## How to apply

First extract image features from the raw H&E image with ExtractFeaturesVisium (VGG16-derived ImageNet features plus RGB quantile matrices) and build the stImage object with LoadImageFeatureVisium from the filtered_feature_bc_matrix.h5 and spatial folder. Run PreProcessing with normalizeMethod='SCT', DimReducMethod='PCA', pcaDim_s=30, pcaDim_i=30, genePercentCut=0.2 and imagePercentCut=0.2, then cluster each modality separately with FindSNNClusters on the 'SCTPCA' and 'ImageFeaturePCA' reductions (dims 1:30, resolutionMax=3). Combine modalities with MultiModalIntegration using MultimodalMethod='WNN', reduction.list=list('SCTPCA','ImageFeaturePCA'), pcaDim_s=30, pcaDim_i=30, and the same resolutionMax; visualize with SpatialDimPlot(group.by='wnn_clusterN') and validate marker co-localization. The rationale is that WNN ignores spatial information, which in the mouse kidney case recovered the glomerulus as a distinct cluster that no spatially aware method achieved.

## Related tools

- **stImage** (R package providing ExtractFeaturesVisium, LoadImageFeatureVisium, PreProcessing, FindSNNClusters and MultiModalIntegration used to run the WNN fusion of SCTPCA and ImageFeaturePCA reductions) — https://github.com/YuWang-VUMC/stImage
- **Seurat** (Underlying framework for SCT normalization, PCA reductions and SNN clustering leveraged by stImage's MultiModalIntegration (WNN)) — https://github.com/satijalab/seurat
- **tidyverse** (R data-manipulation/plotting dependency used in the stImage workflow)

## Examples

```
stImage::MultiModalIntegration(object = st_obj, MultimodalMethod = 'WNN', reduction.list = list('SCTPCA', 'ImageFeaturePCA'), pcaDim_s = 30, pcaDim_i = 30, resolutionMax = 3)
```

## Evaluation signals

- The WNN spatial cluster plot shows a distinct glomerulus cluster co-localized with Podxl/Synpo/Nphs1/Ehd3 feature-plot signal, as in the paper's Figure 6
- Arterial clusters in the WNN result co-localize with Mgp/Flna/Tagln/Eln signal
- The glomerulus substructure is recovered as a distinct cluster only in the WNN-integrated clustering, not in SCT-only, image-only, or spatially aware clusterings
- Both modality reductions ('SCTPCA' and 'ImageFeaturePCA') were computed with 30 dims and clustering used resolutionMax=3 before integration

## Limitations

- No single integration approach performed well on every dataset tested; WNN's success is dataset-dependent
- Multi-modal integration is computationally more intensive and algorithm-dependent, though stImage runtime remains manageable on a typical office desktop
- WNN does not use spatial information, so it may not suit analyses where spatial contiguity is the primary signal

## Evidence

- [other] The WNN strategy, a modality integration method that does not use spatial information, yielded the most accurate identification of kidney components and uniquely identified the glomerulus substructure as a distinct cluster: [evidence span withheld — anchor status: not_found]
- [other] Combine modalities with MultiModalIntegration using MultimodalMethod = 'WNN': [evidence span withheld — anchor status: not_found]
- [methods] MultiModalIntegration is a stImage methods workflow step: [evidence span withheld — anchor status: not_found]
- [discussion] No single integration approach performed well on every dataset tested: "As we expected, there was no single approach performing well in every dataset."
- [readme] stImage is an R package for integrated analysis of spatial transcriptomics and images: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images. StImage is freely available at https://github.com/YuWang-VUMC/stImage."
