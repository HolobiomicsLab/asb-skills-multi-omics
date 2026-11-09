---
name: multi-modal-integration-analysis
description: Use when spatial transcriptomics data contain both histology image features
  and gene expression, and the goal is joint clustering of spots across modalities,
  e.g. comparing WNN vs MCIA vs tICA vs Spectrum against annotated tissue regions.
when_to_use_negative:
- When the histological image is uninformative or conflicts with gene expression,
  or samples are spatially uniform — the paper recommends skipping modality integration
  and using only a spatially aware process
- When neither image features nor spatial structure are informative — rely solely
  on gene expression with standard PCA instead
- When there is only a single modality (gene expression only) — single-modality FindSNNClusters
  on SCTPCA is the appropriate route
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: stImage
  role: Provides LoadImageFeature, PreProcessing, FindSNNClusters, SpatialAwareProcess,
    and MultiModalIntegration (WNN/MCIA/tICA/Spectrum) used to build and integrate
    the modalities
  repo: https://github.com/YuWang-VUMC/stImage
- name: Seurat
  role: Underlying framework for SCT normalization, PCA reductions, and SNN clustering
    on which stImage operates
  repo: https://satijalab.org/seurat
- name: SpatialPCA
  role: Spatially aware dimensionality reduction (slot 'counts', pcaDim_g=20, pcaDim_i=20,
    percentCut 0.25/0.2) whose reductions feed WNN/Spectrum integration; SpatialPCA+WNN
    was the top strategy (ARI 0.62)
  repo: https://github.com/shangll123/SpatialPCA
- name: BayesSpace
  role: Bayesian spatial clustering (assays SCT and ImageFeature, slot 'data') whose
    graphs are combined with WNN and Spectrum integration
- name: tidyverse
  role: R data manipulation used to assemble and compare ARI results across strategies
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:d1d32d65bacbc6c42daa7ad71098fd0858a522447d17614079a965ae79d19b03
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
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# multi-modal-integration-analysis

## Summary

Integrates gene-expression PCA, image-feature PCA, and spatially aware embeddings from spatial transcriptomics data using stImage's MultiModalIntegration function (WNN, MCIA, tICA, Spectrum) to cluster spots and benchmark agreement with pathologist annotations via ARI.

## When to use

Use when spatial transcriptomics data contain both histology image features and gene expression, and the goal is joint clustering of spots across modalities, e.g. comparing WNN vs MCIA vs tICA vs Spectrum against annotated tissue regions.

## When NOT to use

- When the histological image is uninformative or conflicts with gene expression, or samples are spatially uniform — the paper recommends skipping modality integration and using only a spatially aware process
- When neither image features nor spatial structure are informative — rely solely on gene expression with standard PCA instead
- When there is only a single modality (gene expression only) — single-modality FindSNNClusters on SCTPCA is the appropriate route

## Inputs

- Gene count matrix (e.g. H1.tsv from Zenodo 10.5281/zenodo.4751624, HER2 ST platform)
- Histology image (e.g. HE_BT24044_D2.jpg) with spot patches of size 200 formatted as X<x>x<y>
- Spot coordinate/position table with pathologist-annotated labels (e.g. H1_labeled_coordinates.tsv)
- RGB quantile matrix for image features
- Pre-existing PCA reductions (SCTPCA and ImageFeaturePCA) or spatially aware objects (BayesSpace graphs, SpatialPCA/stLearn reductions)

## Outputs

- Cluster assignments (nCluster=6) for spots under each integration strategy (WNN, MCIA, tICA, Spectrum)
- Adjusted Rand index (ARI) per strategy versus ground-truth/pathologist labels
- Results table (e.g. ari_benchmark_results.csv or HER2_H1_ARI_strategies.csv)

## How to apply

Build the stImage object with LoadImageFeature (count table, image features, position table, RGBquantile), then run PreProcessing with SCT normalization, PCA dims 1:20 for gene (pcaDim_s=20) and image (pcaDim_i=20) reductions, and pcaDim_c=6. Optionally first run SpatialAwareProcess (BayesSpace, SpatialPCA, stLearn) and then feed the resulting graphs/reductions into MultiModalIntegration over the SCTPCA + ImageFeaturePCA reductions (or the spatially aware ones) with MultimodalMethod = c('WNN','MCIA','tICA','Spectrum') and nCluster=6. Evaluate each strategy by computing the adjusted Rand index of its cluster assignment against pathologist-annotated labels (e.g. H1_labeled_coordinates.tsv) — in the HER2 H1 benchmark, SpatialPCA followed by WNN achieved the highest ARI (0.62), with ordering Gene/Image ~0.43-0.44 < WNN 0.58 < SpatialPCA+WNN 0.62. Note the paper's decision guidance: apply spatial aware process then modality integration when image features align with gene expression, but skip modality integration when the image is uninformative or spatially uniform.

## Related tools

- **stImage** (Provides LoadImageFeature, PreProcessing, FindSNNClusters, SpatialAwareProcess, and MultiModalIntegration (WNN/MCIA/tICA/Spectrum) used to build and integrate the modalities) — https://github.com/YuWang-VUMC/stImage
- **Seurat** (Underlying framework for SCT normalization, PCA reductions, and SNN clustering on which stImage operates) — https://satijalab.org/seurat
- **SpatialPCA** (Spatially aware dimensionality reduction (slot 'counts', pcaDim_g=20, pcaDim_i=20, percentCut 0.25/0.2) whose reductions feed WNN/Spectrum integration; SpatialPCA+WNN was the top strategy (ARI 0.62)) — https://github.com/shangll123/SpatialPCA
- **BayesSpace** (Bayesian spatial clustering (assays SCT and ImageFeature, slot 'data') whose graphs are combined with WNN and Spectrum integration)
- **tidyverse** (R data manipulation used to assemble and compare ARI results across strategies)

## Examples

```
stImage::MultiModalIntegration(PCA_object, MultimodalMethod = c('WNN','MCIA','tICA','Spectrum'), pcaDim_s = 20, pcaDim_i = 20, nCluster = 6)
```

## Evaluation signals

- ARI ordering matches the reported benchmark: Gene/Image ~0.43-0.44 < SpatialPCA/stLearn ~0.45-0.48 < WNN 0.58 < SpatialPCA+WNN 0.62, with BayesSpace 0.29 and Spectrum/tICA 0.4-0.41
- Integration strategies (WNN, Spectrum) outperform single-modality baselines (gene-only, image-only) on ARI
- Every strategy yields exactly nCluster=6 cluster labels aligned to the annotated spot coordinate table
- MultiModalIntegration runs without error on both the PCA object (SCTPCA + ImageFeaturePCA) and the spatially aware objects (BayesSpace graphs, SpatialPCA/stLearn reductions)
- Results CSV (e.g. HER2_H1_ARI_strategies.csv) contains one ARI row per integrative strategy tested

## Limitations

- No single integration approach performed well on every dataset — strategy choice is dataset-dependent and should be guided by stImage's diagnostic graphs
- Multi-modal integration is computationally more intensive and algorithm-dependent, though runtime remains manageable on a typical office desktop
- The 21 ring pattern genes must be removed from the HER2 count matrix before analysis; spot names must be formatted as X<x>x<y> with patch size 200
- Supplementary Fig. S13/Table S1 (four scenarios guiding strategy choice) are referenced but their content is not provided in the main text

## Evidence

- [abstract] By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information.: "By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information."
- [discussion] we recommend applying a spatial aware process followed by modality integration to fully leverage the complementary information: "we recommend applying a spatial aware process followed by modality integration to fully leverage the complementary information"
- [results] Remove the 21 ring pattern genes from the count matrix and build the stImage object with LoadImageFeature (countTable, imageFeatures, positionTable, RGBquantile).: [evidence span withheld — anchor status: not_found]
- [readme] stImage is freely available at https://github.com/YuWang-VUMC/stImage.: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images. StImage is freely available at https://github.com/YuWang-VUMC/stImage."
