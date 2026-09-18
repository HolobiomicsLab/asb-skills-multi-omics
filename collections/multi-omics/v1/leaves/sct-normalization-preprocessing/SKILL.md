---
name: sct-normalization-preprocessing
description: Use when starting stImage analysis of a Visium or ST spatial transcriptomics
  dataset whose raw counts (e.g. filtered_feature_bc_matrix.h5) and H&E image-derived
  features need normalization, batch/feature filtering, and dimensionality reduction
  before clustering or modality integration.
when_to_use_negative:
- When the histological image is uninformative or conflicts with gene expression —
  the article advises skipping modality integration and using only the spatially aware
  process, in which case paired SCTPCA/ImageFeaturePCA preprocessing for integration
  is unnecessary.
- When neither image features nor spatial structure are informative — rely solely
  on gene expression data with standard PCA rather than stImage's dual-modality SCT
  preprocessing.
- When the input is already a normalized, dimension-reduced Seurat object (e.g. carried
  over from a prior SCTransform run) — re-running SCT normalization risks double normalization.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0610
tools:
- name: stImage
  role: R package providing the PreProcessing, LoadImageFeature/LoadImageFeatureVisium
    and FindSNNClusters functions that execute SCT normalization, dual-modality PCA
    and downstream clustering
  repo: https://github.com/YuWang-VUMC/stImage
- name: Seurat
  role: Underlying R toolkit whose Seurat object structure, SCTransform normalization
    and PCA/DimReduc machinery stImage's PreProcessing wraps; also supplies SpatialDimPlot/SpatialFeaturePlot
    for verification
- name: tidyverse
  role: Data-manipulation packages loaded in the stImage workflow for handling spot-level
    metadata and feature tables during preprocessing
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:e8a3f441a8e0cde39f3e73a6a9a919a472d4a4da2310a6238212f07e415ec6d1
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3076923076923077
n_claims: 13
---

# sct-normalization-preprocessing

## Summary

Run stImage's PreProcessing step with SCTransform-based normalization and PCA on both the gene-expression modality and the deep histology image-feature modality, producing matched SCTPCA and ImageFeaturePCA reductions (default 30 dims each) that feed downstream spatial clustering and modality integration.

## When to use

Use when starting stImage analysis of a Visium or ST spatial transcriptomics dataset whose raw counts (e.g. filtered_feature_bc_matrix.h5) and H&E image-derived features need normalization, batch/feature filtering, and dimensionality reduction before clustering or modality integration.

## When NOT to use

- When the histological image is uninformative or conflicts with gene expression — the article advises skipping modality integration and using only the spatially aware process, in which case paired SCTPCA/ImageFeaturePCA preprocessing for integration is unnecessary.
- When neither image features nor spatial structure are informative — rely solely on gene expression data with standard PCA rather than stImage's dual-modality SCT preprocessing.
- When the input is already a normalized, dimension-reduced Seurat object (e.g. carried over from a prior SCTransform run) — re-running SCT normalization risks double normalization.

## Inputs

- stImage object built via LoadImageFeature/LoadImageFeatureVisium containing (a) a counts matrix from filtered_feature_bc_matrix.h5 (10x Visium or ST platform), (b) VGG16-derived ImageNet deep histology features and RGB quantile matrices extracted from the raw H&E image
- Normalization/reduction parameters: normalizeMethod = 'SCT', DimReducMethod = 'PCA', pcaDim_s, pcaDim_i, genePercentCut, imagePercentCut

## Outputs

- Seurat/stImage object with an 'SCTPCA' reduction (normalized gene-expression PCA, default 30 dims)
- Seurat/stImage object with an 'ImageFeaturePCA' reduction (image-feature PCA, default 30 dims)
- Feature-filtered expression and image-feature matrices with low-prevalence genes/features and 21 ring pattern genes removed

## How to apply

After building the stImage object with LoadImageFeatureVisium/LoadImageFeature (extracting VGG16-derived ImageNet image features and RGB quantile matrices from the raw H&E image), call PreProcessing with normalizeMethod = 'SCT' so the expression matrix is SCTransform-normalized, and DimReducMethod = 'PCA'. Set matched dimensions for both modalities (pcaDim_s = 30 for the transcriptomic SCTPCA reduction and pcaDim_i = 30 for the image-feature ImageFeaturePCA reduction) and apply equal feature-retention cutoffs of genePercentCut = 0.2 and imagePercentCut = 0.2 so that only features detected in at least 20% of spots are kept in each modality. The stImage preprocessing also removes the 21 ring pattern genes mentioned in the original paper to suppress tissue-edge artifacts before reduction. The resulting paired SCTPCA and ImageFeaturePCA reductions are the required inputs for FindSNNClusters on each modality and for MultiModalIntegration (e.g. WNN with pcaDim_s = 30, pcaDim_i = 30). Seurat's spatial toolkit (SpatialDimPlot, SpatialFeaturePlot) then verifies the reduced dimensions still align with tissue anatomy.

## Related tools

- **stImage** (R package providing the PreProcessing, LoadImageFeature/LoadImageFeatureVisium and FindSNNClusters functions that execute SCT normalization, dual-modality PCA and downstream clustering) — https://github.com/YuWang-VUMC/stImage
- **Seurat** (Underlying R toolkit whose Seurat object structure, SCTransform normalization and PCA/DimReduc machinery stImage's PreProcessing wraps; also supplies SpatialDimPlot/SpatialFeaturePlot for verification)
- **tidyverse** (Data-manipulation packages loaded in the stImage workflow for handling spot-level metadata and feature tables during preprocessing)

## Examples

```
stImage::PreProcessing(obj, normalizeMethod = 'SCT', DimReducMethod = 'PCA', pcaDim_s = 30, pcaDim_i = 30, genePercentCut = 0.2, imagePercentCut = 0.2)
```

## Evaluation signals

- Both reductions (SCTPCA and ImageFeaturePCA) exist on the stImage object with exactly 30 dimensions each (pcaDim_s = 30, pcaDim_i = 30).
- genePercentCut = 0.2 and imagePercentCut = 0.2 were applied: spot-level feature-detection matrices show no gene or image feature detected in fewer than 20% of spots.
- The 21 ring pattern genes from the original paper are absent from the preprocessed expression matrix.
- Downstream FindSNNClusters and MultiModalIntegration (WNN) succeed using the SCTPCA and ImageFeaturePCA reductions as reduction.list inputs.
- SpatialDimPlot on the WNN clusters and AllSpatialFeaturePlot of markers (e.g. Podxl, Synpo for glomeruli) confirm the reduced space still co-localizes with known tissue structures, as in the paper's Figure 6.

## Limitations

- Multi-modal integration after preprocessing is computationally more intensive and algorithm-dependent, though stImage's overall runtime remains manageable on a typical office desktop (Supplementary Table S2).
- No single integration approach performed well on every dataset tested — the SCT-normalized dual-modality reductions are not a universal solution when spatial homogeneity or image informativeness assumptions are violated.
- The choice of whether to proceed to modality integration after SCT preprocessing should be guided by stImage's diagnostic graphs; Supplementary Fig. S13 and Table S1 detailing the four guiding scenarios are referenced but not reproduced in the article text.

## Evidence

- [methods] Ring-pattern gene removal is part of stImage preprocessing: "remove the 21 ring pattern genes mentioned in the orginal paper"
- [results] Source-card workflow grounds the exact SCT/PCA parameters: [evidence span withheld — anchor status: not_found]
- [abstract] Deep histology feature generation is the basis for the image-feature PCA: "By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information."
- [intro] No universal one-size-fits-all solution — context for when_not_to_use: "Unfortunately, these requirements are not always met, rendering a universal, one-size-fits-all solution elusive."
