---
name: single-modality-vs-integration-performance-analysis
description: Use when decision support for choosing among stImage's 54 integrative
  strategies when ground-truth cluster labels (e.g., from simulated ST data) are available
  and a quantitative ARI comparison of gene-only, image-only, spatially aware, and
  integrated clusterings is needed.
when_to_use_negative:
- When the histological image is uninformative or conflicts with gene expression —
  the article recommends skipping modality integration and only employing a spatially
  aware process
- When spatial heterogeneity in tissue structures is prominent — the article recommends
  skipping the spatial aware process and proceeding directly to modality integration
  rather than benchmarking both
- When neither image features nor spatial structure are informative — rely solely
  on gene expression with standard PCA instead
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: stImage
  role: Provides LoadImageFeature, PreProcessing, FindSNNClusters, SpatialAwareProcess,
    and MultiModalIntegration used to generate every strategy's clustering
  repo: github.com/YuWang-VUMC/stImage
- name: Seurat
  role: Single-cell/spatial R toolkit underlying the object model and SCT normalization
    used for the gene and image assays
  repo: https://satijalab.org/seurat
- name: SpatialPCA
  role: Spatially aware dimensionality-reduction method invoked inside SpatialAwareProcess
    with pcaDim_g = pcaDim_i = 20
  repo: github.com/shangll123/SpatialPCA
- name: BayesSpace
  role: Bayesian spatial clustering backend used in SpatialAwareProcess with platform
    = 'ST'
  repo: https://bioconductor.org/packages/BayesSpace
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:58097c80f7030bd712df3d050eed49623020912162179709a5000f58c7cb1997
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.14285714285714285
n_claims: 7
---

# single-modality-vs-integration-performance-analysis

## Summary

Benchmarks stImage's single-modality strategies (FindSNNClusters on SCTPCA/ImageFeaturePCA, SpatialAwareProcess) against multi-modal integration methods (WNN, MCIA, tICA, Spectrum) on simulated spatial transcriptomics data with known ground-truth labels, quantifying clustering agreement via Adjusted Rand Index.

## When to use

Decision support for choosing among stImage's 54 integrative strategies when ground-truth cluster labels (e.g., from simulated ST data) are available and a quantitative ARI comparison of gene-only, image-only, spatially aware, and integrated clusterings is needed.

## When NOT to use

- When the histological image is uninformative or conflicts with gene expression — the article recommends skipping modality integration and only employing a spatially aware process
- When spatial heterogeneity in tissue structures is prominent — the article recommends skipping the spatial aware process and proceeding directly to modality integration rather than benchmarking both
- When neither image features nor spatial structure are informative — rely solely on gene expression with standard PCA instead

## Inputs

- Simulated spatial transcriptomics gene expression matrix
- Image feature matrix and RGB quantile matrix
- Ground-truth simulated cluster labels
- Target number of simulated clusters

## Outputs

- Per-strategy cluster label assignments
- ARI table of all strategies vs. ground-truth labels
- ari_benchmark_results.csv

## How to apply

Load the simulated ST data (gene expression, image feature matrix, RGB quantile matrix, ground-truth labels) into stImage via LoadImageFeature, then normalize and reduce each modality with PreProcessing (SCT normalization, PCA with 20 dims for gene and image assays, pcaDim_c = 6). Obtain single-modality baselines with FindSNNClusters on the SCTPCA and ImageFeaturePCA reductions at the simulated cluster number, and spatially aware baselines with SpatialAwareProcess using BayesSpace (platform = 'ST') and SpatialPCA (pcaDim_g = pcaDim_i = 20). Run MultiModalIntegration with MultimodalMethod = c('WNN','MCIA','tICA','Spectrum') over the SCT and ImageFeature PCA reductions (genePercentCut = 0.25, imagePercentCut = 0.2, pcaDim_s = pcaDim_i = 20), and also combine the spatially aware graphs with WNN and Spectrum. Compute ARI of every strategy's labels against ground truth and write ari_benchmark_results.csv.

## Related tools

- **stImage** (Provides LoadImageFeature, PreProcessing, FindSNNClusters, SpatialAwareProcess, and MultiModalIntegration used to generate every strategy's clustering) — github.com/YuWang-VUMC/stImage
- **Seurat** (Single-cell/spatial R toolkit underlying the object model and SCT normalization used for the gene and image assays) — https://satijalab.org/seurat
- **SpatialPCA** (Spatially aware dimensionality-reduction method invoked inside SpatialAwareProcess with pcaDim_g = pcaDim_i = 20) — github.com/shangll123/SpatialPCA
- **BayesSpace** (Bayesian spatial clustering backend used in SpatialAwareProcess with platform = 'ST') — https://bioconductor.org/packages/BayesSpace

## Examples

```
obj <- stImage::MultiModalIntegration(object = obj, MultimodalMethod = c('WNN','MCIA','tICA','Spectrum'), genePercentCut = 0.25, imagePercentCut = 0.2, pcaDim_s = 20, pcaDim_i = 20)
```

## Evaluation signals

- ari_benchmark_results.csv is produced with one ARI value per strategy, all computed against the simulated ground-truth labels
- Integration strategies (WNN, Spectrum, MCIA, tICA) yield higher ARI than gene-only and image-only FindSNNClusters baselines
- Under high dropout rates, WNN and Spectrum achieve significantly higher ARI than MCIA and tICA
- PCA dimensions match the reported settings (20 dims for gene and image assays, pcaDim_c = 6, pcaDim_s = pcaDim_i = 20)
- Single-modality clusterings use the same simulated cluster number as the ground truth

## Limitations

- No single integration approach performed well on every dataset; the simulated-data ordering may not transfer to real tissues
- Multi-modal integration is computationally more intensive and algorithm-dependent, though stImage runtime remains manageable on a typical office desktop
- Supplementary Fig. S13/Table S1 (the four scenarios guiding strategy choice) are referenced but their content is not provided, limiting guidance for real-data strategy selection

## Evidence

- [abstract] 54 integrative strategies: "By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information."
- [intro] compare methods and use diagnostic graphs: "stImage enables users to apply and compare different methods simultaneously, as well as make data-driven decisions using its diagnostic graphs that reflect the spatial and histological"
- [discussion] spatial aware + integration recommendation: "we recommend applying a spatial aware process followed by modality integration to fully leverage the complementary information"
- [discussion] no universal winner: "As we expected, there was no single approach performing well in every dataset."
- [other] integration call: [evidence span withheld — anchor status: not_found]
- [readme] package purpose: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images."
