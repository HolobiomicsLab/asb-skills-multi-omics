---
name: spatially-aware-dimensionality-reduction
description: Use when analyzing spatial transcriptomics data (e.g., Visium DLPFC counts
  with spot coordinates) and you need dimensionality reduction that incorporates spatial
  location, or a spatially-aware baseline to benchmark modality integration against.
when_to_use_negative:
- When spatial heterogeneity in tissue structures is prominent, the article recommends
  skipping the spatially-aware step and proceeding directly to modality integration
- When neither image features nor spatial structure are informative, rely solely on
  gene expression with standard methods such as PCA instead of spatially-aware reduction
- When input lacks spot spatial coordinates or the platform is not compatible with
  BayesSpace's platform='ST' settings
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: stImage
  role: R framework whose SpatialAwareProcess, PreProcessing, and FindSNNClusters
    functions implement the spatially-aware processing and clustering workflow
  repo: https://github.com/YuWang-VUMC/stImage
- name: SpatialPCA
  role: Spatially-aware dimensionality reduction method applied on raw counts (slot='counts',
    pcaDim_g=20, pcaDim_i=20) within SpatialAwareProcess
  repo: https://github.com/shangll123/SpatialPCA
- name: BayesSpace
  role: Bayesian spatial clustering/refinement applied on SCT and ImageFeature assays
    (slot 'data', platform='ST') within SpatialAwareProcess
- name: Seurat
  role: Underlying single-cell/spatial toolkit for the SCT normalization, PCA (SCTPCA),
    and SNN clustering used as gene-only baseline
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:1fb58be23f0fa92a0f5f43aae13e4fa0331bf83786fe88964ee542c70ee192b6
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# spatially-aware-dimensionality-reduction

## Summary

Applies spatially-aware dimensionality reduction within the stImage framework (SpatialPCA and BayesSpace via SpatialAwareProcess) so that spot neighborhood information informs low-dimensional representations before clustering, providing stronger baselines (e.g., GeneSpatialPCA ~0.43 ARI, GeneBayesSpace ~0.56 ARI on DLPFC 151673) than gene-only PCA.

## When to use

Use when analyzing spatial transcriptomics data (e.g., Visium DLPFC counts with spot coordinates) and you need dimensionality reduction that incorporates spatial location, or a spatially-aware baseline to benchmark modality integration against.

## When NOT to use

- When spatial heterogeneity in tissue structures is prominent, the article recommends skipping the spatially-aware step and proceeding directly to modality integration
- When neither image features nor spatial structure are informative, rely solely on gene expression with standard methods such as PCA instead of spatially-aware reduction
- When input lacks spot spatial coordinates or the platform is not compatible with BayesSpace's platform='ST' settings

## Inputs

- Visium spatial transcriptomics gene expression count matrix (e.g., DLPFC sample 151673 from spatialLIBD)
- Spot spatial coordinates (spatial locations)
- H&E histology image-derived features (ImageFeature PCA and RGB PCA reductions from stImage::ExtractFeatures, e.g., VGG16 patchSize 200)
- SCT-normalized expression (SCT assay) and gene-only PCA reduction (dims 1:20)
- Manual layer annotations (ground-truth labels for ARI validation)

## Outputs

- Spatially-aware low-dimensional embeddings/representations (SpatialPCA PCs; BayesSpace distance and similarity graphs)
- Per-spot spatially-aware cluster labels (via FindSNNClusters at the target cluster number)
- ARI comparison table quantifying clustering accuracy vs manual annotations (e.g., dlpfc_151673_ari.csv)

## How to apply

Build the stImage object and run PreProcessing with normalizeMethod='SCT', DimReducMethod='PCA', pcaDim_s=20, genePercentCut=0.25, imagePercentCut=0.2, then call SpatialAwareProcess using BayesSpace (platform='ST', assays 'SCT' and 'ImageFeature', slot 'data') and SpatialPCA (slot='counts', pcaDim_g=20, pcaDim_i=20, percentCut 0.25/0.2). Cluster the spatially-aware reductions with FindSNNClusters at the target cluster number (e.g., 7 DLPFC layers) and retain the gene-only SCTPCA (dims 1:20) clustering as the non-spatial baseline. The rationale is that SpatialPCA models the spatial correlation structure across tissue locations during dimensionality reduction, while BayesSpace leverages spatial neighborhood information through Bayesian strategies for clustering; benchmarking on DLPFC 151673 against manual layer annotations shows this raises ARI above gene-only PCA. As the article's decision rule notes, apply the spatially-aware step (optionally followed by modality integration) when image features align with gene expression, and use it alone—skipping integration—when the histological image is uninformative or conflicts with expression.

## Related tools

- **stImage** (R framework whose SpatialAwareProcess, PreProcessing, and FindSNNClusters functions implement the spatially-aware processing and clustering workflow) — https://github.com/YuWang-VUMC/stImage
- **SpatialPCA** (Spatially-aware dimensionality reduction method applied on raw counts (slot='counts', pcaDim_g=20, pcaDim_i=20) within SpatialAwareProcess) — https://github.com/shangll123/SpatialPCA
- **BayesSpace** (Bayesian spatial clustering/refinement applied on SCT and ImageFeature assays (slot 'data', platform='ST') within SpatialAwareProcess)
- **Seurat** (Underlying single-cell/spatial toolkit for the SCT normalization, PCA (SCTPCA), and SNN clustering used as gene-only baseline) — https://github.com/satijalab/seurat

## Examples

```
st_obj <- SpatialAwareProcess(st_obj, method='SpatialPCA', slot='counts', pcaDim_g=20, pcaDim_i=20, genePercentCut=0.25, imagePercentCut=0.2)
```

## Evaluation signals

- On Visium DLPFC sample 151673, computed ARI must reproduce the benchmark ordering: GeneBayesSpace ~0.56 > GeneSpatialPCA ~0.43 > gene-only ~0.36 > image-only ~0.18
- Spatially-aware clusterings must use the specified dimensionalities (pcaDim_g=20, pcaDim_i=20, dims 1:20) and prefilter/filter cutoffs (genePercentCut=0.25, imagePercentCut=0.2)
- BayesSpace runs must be configured with platform='ST' and operate on the 'SCT' and 'ImageFeature' assays with slot 'data'; SpatialPCA on slot 'counts'
- ARI must be computed against the manual layer annotations of sample 151673, and the full comparison table written (e.g., dlpfc_151673_ari.csv)
- Clustering must use FindSNNClusters at the target cluster number (7 cortical layers for DLPFC)

## Limitations

- No single approach performed well on every dataset: the spatially-aware-plus-integration path helps only when image features align well with gene expression, and the article recommends skipping modality integration (keeping only the spatially-aware process) when the image is uninformative or conflicts with expression
- Multi-modal and spatially-aware processing is computationally more intensive and algorithm-dependent, though stImage runtime remains manageable on a typical office desktop
- stImage depends on SPARK/SPARKX, SpatialPCA, Bioconductor packages (rhdf5, SingleCellExperiment, BayesSpace, omicade4), and a working tensorflow/keras installation

## Evidence

- [intro] SpatialPCA models spatial correlation during dimensionality reduction: "SpatialPCA [11] models the spatial correlation structure across tissue locations during dimensionality reduction"
- [intro] BayesSpace leverages spatial neighborhoods via Bayesian strategies: "BASS [16] and BayesSpace [14] leverage spatial neighborhood information through Bayesian statistical strategies for resolution enhancement or clustering analysis"
- [discussion] Recommendation: spatial-aware process followed by modality integration when features align: "we recommend applying a spatial aware process followed by modality integration to fully leverage the complementary information"
- [discussion] Recommendation: keep only the spatially-aware process when image is uninformative: "skipping the modality integration and only employing a spatially aware process can be advantageous"
- [readme] stImage is an R package for integrated analysis of ST and modality images; SpatialPCA is a dependency: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images."
