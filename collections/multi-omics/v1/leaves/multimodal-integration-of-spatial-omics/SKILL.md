---
name: multimodal-integration-of-spatial-omics
description: Use when a Visium/ST dataset has H&E image data with informative morphology
  (e.g. DLPFC layers or HER2 tumor regions) and you need joint gene+image+space spot
  clusters to compare against manual annotations.
when_to_use_negative:
- When the histological image is uninformative or conflicts with gene expression —
  the paper advises skipping modality integration and only employing a spatially aware
  process
- When samples are spatially uniform with no informative image features — rely solely
  on gene expression data using standard methods such as PCA
- When the goal is sub-spot resolution or >2 spatial omics modalities — stImage integration
  was demonstrated on spot-level Visium/ST gene+image pairs and is only theoretically
  extensible to 3D or extra modalities
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3391
tools:
- name: stImage
  role: R package providing LoadImageFeature, PreProcessing, FindSNNClusters and MultiModalIntegration
    (WNN, MCIA, tICA, Spectrum) for gene+image+space integration
  repo: https://github.com/YuWang-VUMC/stImage
- name: Seurat
  role: single-cell/spatial toolkit underlying stImage's SNN clustering and weighted-nearest-neighbor
    multimodal framework
  repo: https://github.com/satijalab/seurat
- name: SpatialPCA
  role: spatially aware dimensionality reduction whose reductions feed WNN/Spectrum
    integration
  repo: https://github.com/shangll123/SpatialPCA
- name: BayesSpace
  role: Bayesian spatial smoothing producing distance/similarity graphs integrated
    with WNN and Spectrum
- name: tidyverse
  role: R data wrangling for building ARI comparison tables across strategies
merged_aliases:
- multimodal-spatial-clustering-integration
merged_alias_records:
- alias: multimodal-spatial-clustering-integration
  slug: multimodal-spatial-clustering-integration
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:4b04ab6983b970ced530c5245292a0597b5dfd815ac427fb8a0adff1a6df5ccf
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
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# multimodal-integration-of-spatial-omics

## Summary

Integrate spatial transcriptomic gene-expression reductions with deep-learning-derived histology image features (and spatially aware reductions) into joint spot clusterings, using stImage's 54 integrative strategies such as WNN, MCIA, tICA and Spectrum.

## When to use

Use when a Visium/ST dataset has H&E image data with informative morphology (e.g. DLPFC layers or HER2 tumor regions) and you need joint gene+image+space spot clusters to compare against manual annotations.

## When NOT to use

- When the histological image is uninformative or conflicts with gene expression — the paper advises skipping modality integration and only employing a spatially aware process
- When samples are spatially uniform with no informative image features — rely solely on gene expression data using standard methods such as PCA
- When the goal is sub-spot resolution or >2 spatial omics modalities — stImage integration was demonstrated on spot-level Visium/ST gene+image pairs and is only theoretically extensible to 3D or extra modalities

## Inputs

- Visium/ST spatial transcriptomics count matrix with spot coordinates and manual spot-type annotations (e.g. spatialLIBD DLPFC sample 151673)
- H&E histology image from which VGG16 (ImageNet) deep features and RGB quantiles are extracted (patchSize 200)
- Per-modality dimensionality reductions (SCTPCA, ImageFeaturePCA, RGBPCA, or SpatialPCA/BayesSpace reductions)

## Outputs

- Joint spot cluster assignments per integration strategy (e.g. 6 WNN clusters in HER2 H1)
- ARI comparison table quantifying each strategy against manual annotations (e.g. dlpfc_151673_ari.csv)
- Region-supported cluster identities (e.g. ERBB2 exclusively expressed in the in-situ cancer WNN cluster)

## How to apply

First cluster each modality separately with stImage: gene expression after PreProcessing (normalizeMethod='SCT', DimReducMethod='PCA', pcaDim_s=20) via SCTPCA, image features extracted with VGG16 (ImageNet, patchSize 200) via ImageFeaturePCA (pcaDim_i=20) and RGB quantiles via RGBPCA. Optionally apply a spatially aware process first — SpatialPCA (slot 'counts', pcaDim_g=20, pcaDim_i=20, percentCuts 0.25/0.2), BayesSpace (platform='ST', slot 'data') or stLearn (cosine distance, weights_matrix_all, percentCut=0.03) — since the paper recommends 'applying a spatial aware process followed by modality integration' when image features align well with gene expression. Then call stImage::MultiModalIntegration with method='WNN', 'MCIA', 'tICA' or 'Spectrum' on the SCTPCA/ImageFeaturePCA reductions (pcaDim_s=20, pcaDim_i=20, genePercentCut=0.25, imagePercentCut=0.2), retaining the gene-only clustering as a baseline. Judge success by computing the adjusted Rand index against manual annotations and comparing integrated vs gene-only ARIs: on DLPFC 151673, WNN integration of image features lowered ARI relative to gene-only (GeneBayesSpace ~0.56 > GeneSpatialPCA ~0.43 > gene-only ~0.36 > image-only ~0.18), while stLearn refinement slightly improved it — so always benchmark integration against gene-only rather than assuming it helps.

## Related tools

- **stImage** (R package providing LoadImageFeature, PreProcessing, FindSNNClusters and MultiModalIntegration (WNN, MCIA, tICA, Spectrum) for gene+image+space integration) — https://github.com/YuWang-VUMC/stImage
- **Seurat** (single-cell/spatial toolkit underlying stImage's SNN clustering and weighted-nearest-neighbor multimodal framework) — https://github.com/satijalab/seurat
- **SpatialPCA** (spatially aware dimensionality reduction whose reductions feed WNN/Spectrum integration) — https://github.com/shangll123/SpatialPCA
- **BayesSpace** (Bayesian spatial smoothing producing distance/similarity graphs integrated with WNN and Spectrum)
- **tidyverse** (R data wrangling for building ARI comparison tables across strategies)

## Examples

```
st_obj <- MultiModalIntegration(st_obj, reduction_s='SCTPCA', reduction_i='ImageFeaturePCA', method='wnn', pcaDim_s=20, pcaDim_i=20, genePercentCut=0.25, imagePercentCut=0.2)
```

## Evaluation signals

- ARI ordering reproduced on DLPFC 151673: GeneBayesSpace ~0.56 > GeneSpatialPCA ~0.43 > gene-only ~0.36 > image-only ~0.18, with image-integrated WNN lowering ARI and stLearn slightly improving it
- Each modality clustering (SCTPCA dims 1:20, ImageFeaturePCA dims 1:20, RGBPCA) must be computed before integration; cluster number must match the target annotation count (e.g. clusterNum = 6 for HER2 H1)
- Integrated clusters should be region-specific: ERBB2 exclusively expressed in the in-situ cancer cluster, TIMP1 in invasive cancer, CFD in adipose, CD74 in immune infiltrate
- stImage object must contain both 'SCT'/'ImageFeature' assays with correctly set slots ('data' for BayesSpace, 'counts' for SpatialPCA) before MultiModalIntegration
- No single integration approach should be assumed optimal — the paper states 'there was no single approach performing well in every dataset', so per-dataset benchmarking is mandatory

## Limitations

- Image integration can hurt performance: on DLPFC, WNN, SpatialPCA+WNN and BayesSpace+WNN all yielded lower ARI than gene-only counterparts
- Multi-modal integration is computationally more intensive and algorithm-dependent, though stImage runtime remains manageable on a typical office desktop
- No single approach performed well on every dataset; stImage's diagnostic graphs guide strategy choice, but the four-scenario guidance (Supplementary Fig. S13 / Table S1) content is not provided in the accessible text
- Requires TensorFlow/Keras for VGG16 feature extraction, adding nontrivial installation dependencies (SPARK/SPARKX, SpatialPCA, BayesSpace, omicade4 via BiocManager)

## Evidence

- [abstract] By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information.: "By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information."
- [discussion] we recommend applying a spatial aware process followed by modality integration to fully leverage the complementary information: "we recommend applying a spatial aware process followed by modality integration to fully leverage the complementary information"
- [discussion] As we expected, there was no single approach performing well in every dataset.: "As we expected, there was no single approach performing well in every dataset."
- [methods] MultiModalIntegration: "MultiModalIntegration"
- [readme] Users should also make sure to successfully install the tensorflow and keras following instruction Tensorflow for R: [evidence span withheld — anchor status: external_doc]
