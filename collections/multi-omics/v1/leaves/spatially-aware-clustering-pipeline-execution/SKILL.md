---
name: spatially-aware-clustering-pipeline-execution
description: Use when spatial transcriptomics data with paired histology images (e.g.
  ST/Visium spots) needs clustering, and you must choose among gene-only, image-only,
  spatially aware, and multi-modal integration strategies or benchmark them against
  known labels.
when_to_use_negative:
- When neither image features nor spatial structure are informative — the article
  advises relying solely on gene expression data using standard methods such as PCA
- When the tissue is spatially homogeneous and the histological image is uninformative
  or conflicts with gene expression — skip modality integration and use only a spatially
  aware process
- When the data is not spatially resolved transcriptomics paired with histology (e.g.
  dissociated single-cell RNA-seq without spatial coordinates)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: stImage
  role: Core R package providing LoadImageFeature, PreProcessing, FindSNNClusters,
    SpatialAwareProcess, and MultiModalIntegration
  repo: github.com/YuWang-VUMC/stImage
- name: Seurat
  role: Single-cell/spatial object infrastructure underlying the SCT and PCA reductions
  repo: https://satijalab.org/seurat
- name: SpatialPCA
  role: Spatially aware dimensionality reduction backend used within SpatialAwareProcess
    (pcaDim_g = pcaDim_i = 20)
  repo: shangll123/SpatialPCA
- name: BayesSpace
  role: Bayesian spatial clustering backend used within SpatialAwareProcess with platform
    = 'ST'
  repo: https://bioconductor.org/packages/BayesSpace
- name: tidyverse
  role: Data manipulation and assembly of the ARI results table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:d27e6303741f052c2ef8ec2f43a94137a44b78a2071631749f8401a5c7202dbd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BayesSpace
  - R
  - Seurat
  - SpatialPCA
  - stImage
  - tidyverse
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# spatially-aware-clustering-pipeline-execution

## Summary

End-to-end stImage workflow that builds a spatial transcriptomics analysis object, normalizes gene and image modalities, runs spatially aware clustering (BayesSpace/SpatialPCA) plus multi-modal integration (WNN, MCIA, tICA, Spectrum), and scores strategies by Adjusted Rand Index.

## When to use

Use when spatial transcriptomics data with paired histology images (e.g. ST/Visium spots) needs clustering, and you must choose among gene-only, image-only, spatially aware, and multi-modal integration strategies or benchmark them against known labels.

## When NOT to use

- When neither image features nor spatial structure are informative — the article advises relying solely on gene expression data using standard methods such as PCA
- When the tissue is spatially homogeneous and the histological image is uninformative or conflicts with gene expression — skip modality integration and use only a spatially aware process
- When the data is not spatially resolved transcriptomics paired with histology (e.g. dissociated single-cell RNA-seq without spatial coordinates)

## Inputs

- Spatial transcriptomics gene expression count matrix (ST or Visium platform)
- Histology image feature matrix (deep learning-derived features)
- RGB quantile matrix extracted from the histology image
- Optional simulated ground-truth cluster labels for ARI evaluation

## Outputs

- Cluster labels per strategy (single-modality, spatially aware, integrated)
- ARI score table comparing all strategies against ground-truth labels (e.g. ari_benchmark_results.csv)
- Diagnostic graph guiding integration strategy choice

## How to apply

Build the analysis object with stImage::LoadImageFeature from the gene expression matrix, image feature matrix, RGB quantile matrix, and any ground-truth labels. Normalize and reduce each modality with stImage::PreProcessing (SCT normalization, PCA with 20 dims for gene and image assays, pcaDim_c = 6), then obtain single-modality baselines with stImage::FindSNNClusters on the SCTPCA and ImageFeaturePCA reductions at the expected cluster number. Run stImage::SpatialAwareProcess per modality using BayesSpace (platform = 'ST') and SpatialPCA (pcaDim_g = pcaDim_i = 20) to inject spatial-neighborhood information, then stImage::MultiModalIntegration with MultimodalMethod = c('WNN','MCIA','tICA','Spectrum') over the SCT and ImageFeature PCA reductions (genePercentCut = 0.25, imagePercentCut = 0.2, pcaDim_s = pcaDim_i = 20), also combining spatially aware graphs with WNN and Spectrum. Compute ARI of each strategy's labels against ground truth and assemble the result table (e.g. ari_benchmark_results.csv). Choose the final strategy from the article's decision logic: prefer spatial aware process followed by modality integration when image features align with gene expression; skip the spatial aware step when spatial heterogeneity is prominent; skip modality integration and use only spatially aware clustering when the image is uninformative or conflicts with expression.

## Related tools

- **stImage** (Core R package providing LoadImageFeature, PreProcessing, FindSNNClusters, SpatialAwareProcess, and MultiModalIntegration) — github.com/YuWang-VUMC/stImage
- **Seurat** (Single-cell/spatial object infrastructure underlying the SCT and PCA reductions) — https://satijalab.org/seurat
- **SpatialPCA** (Spatially aware dimensionality reduction backend used within SpatialAwareProcess (pcaDim_g = pcaDim_i = 20)) — shangll123/SpatialPCA
- **BayesSpace** (Bayesian spatial clustering backend used within SpatialAwareProcess with platform = 'ST') — https://bioconductor.org/packages/BayesSpace
- **tidyverse** (Data manipulation and assembly of the ARI results table)

## Examples

```
library(devtools); install_github("YuWang-VUMC/stImage"); stImage::SpatialAwareProcess(obj, method = "SpatialPCA", platform = "ST", pcaDim_g = 20, pcaDim_i = 20)
```

## Evaluation signals

- Integrated strategies (WNN, Spectrum) achieve significantly higher ARI than MCIA, tICA, and the gene-only/image-only baselines under high dropout rates
- Every strategy's clustering is evaluable via ARI against the simulated ground-truth labels; ARI ordering matches the benchmark (integration > single-modality)
- PreProcessing yields SCTPCA (20 dims) and ImageFeaturePCA (20 dims) reductions on both assays, with pcaDim_c = 6 for the combined space
- Cluster count passed to FindSNNClusters matches the simulated cluster number
- Results table written to ari_benchmark_results.csv with one row per strategy and matching label/ARI columns

## Limitations

- No single integration approach performed well on every dataset tested — strategy choice is dataset-dependent and should follow the diagnostic graph
- Multi-modal integration is computationally more intensive and algorithm-dependent, though stImage runtime remains manageable on a typical office desktop
- Missing signals: Supplementary Fig. S13 and Table S1 (the four strategy-choice scenarios) and Supplementary Table S2 (runtime/resource usage) are referenced but their content is not provided
- Installing stImage requires non-CRAN dependencies (SPARK/SPARKX, SpatialPCA via devtools, Bioconductor packages rhdf5/SingleCellExperiment/BayesSpace/omicade4) plus a working tensorflow/keras Python environment

## Evidence

- [abstract] By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information.: "By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information."
- [methods] Run stImage::MultiModalIntegration with MultimodalMethod = c('WNN', 'MCIA', 'tICA', 'Spectrum') over the SCT and ImageFeature PCA reductions: [evidence span withheld — anchor status: not_found]
- [discussion] we recommend applying a spatial aware process followed by modality integration to fully leverage the complementary information: "we recommend applying a spatial aware process followed by modality integration to fully leverage the complementary information"
- [discussion] it is preferable to skip the spatial aware process and proceed directly to modality integration: "it is preferable to skip the spatial aware process and proceed directly to modality integration"
- [readme] install_github('xzhoulab/SPARK'): "install_github('xzhoulab/SPARK')
install_github("shangll123/SpatialPCA")"
