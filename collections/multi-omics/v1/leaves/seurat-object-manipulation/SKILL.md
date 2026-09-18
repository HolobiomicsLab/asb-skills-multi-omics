---
name: seurat-object-manipulation
description: Use when a spatial transcriptomics analysis must be carried out on a
  Seurat object so that stImage functions (PreProcessing, extractNeighborDist, diagnostic
  plots) can access its spatial assay and reductions.
when_to_use_negative:
- Input is raw counts or a plain expression matrix that has not yet been organized
  into a Seurat spatial object with assay, reductions, and spatial coordinates
- Analysis targets non-spatial single-cell RNA-seq with no spatial assay or histology
  image modality
- The goal is a stImage function that operates on its own output (e.g. plotting from
  the extractNeighborDist result) rather than on the Seurat object itself
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Seurat
  role: Provides the R object framework (spatial assay, reductions, metadata) that
    stImage reads and manipulates
  repo: https://satijalab.org/seurat
- name: stImage
  role: Consumes the Seurat object in PreProcessing and extractNeighborDist and returns
    results as Seurat-compatible objects/plots
  repo: https://github.com/YuWang-VUMC/stImage
- name: R
  role: Runtime environment in which Seurat objects are loaded and passed to stImage
    functions
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:8a41cf8673fc2e042359cc060c88408d6665af354c2610774eb0234c6bc6e1fc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - stImage
  - tidyverse
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3
n_claims: 10
---

# seurat-object-manipulation

## Summary

Skill for loading, inspecting, and passing Seurat spatial objects (e.g. the SCT-normalized HER2 Spatial Transcriptomics object with gene and image-feature PCA reductions) into stImage's preprocessing and diagnostic functions.

## When to use

Use when a spatial transcriptomics analysis must be carried out on a Seurat object so that stImage functions (PreProcessing, extractNeighborDist, diagnostic plots) can access its spatial assay and reductions.

## When NOT to use

- Input is raw counts or a plain expression matrix that has not yet been organized into a Seurat spatial object with assay, reductions, and spatial coordinates
- Analysis targets non-spatial single-cell RNA-seq with no spatial assay or histology image modality
- The goal is a stImage function that operates on its own output (e.g. plotting from the extractNeighborDist result) rather than on the Seurat object itself

## Inputs

- Seurat object with SCT-normalized spatial assay (e.g. HER2 breast cancer ST data)
- Gene-expression PCA reduction (20 PCs)
- Image-feature PCA reduction (20 PCs) from deep learning-derived histology features
- Spot spatial coordinates metadata

## Outputs

- Validated Seurat object ready for stImage diagnostics
- Neighbor-distance object from extractNeighborDist (per-spot neighbor vs non-neighbor similarities)
- Diagnostic ggplot PDFs (e.g. HER2_SimilarToNeighbors.pdf, HER2_DistConsistency.pdf)

## How to apply

Load or construct the Seurat object so it contains the SCT-normalized spatial assay plus two PCA reductions (gene-expression PCA and image-feature PCA, 20 PCs), as produced by the stImage PreProcessing step on the HER2 breast cancer Spatial Transcriptomics dataset. Verify the object carries spot-level spatial coordinates and image features before passing it to stImage's extractNeighborDist, which computes per-spot similarity/distance to spatial neighbors versus non-neighbors in the top-PC embedding space. The resulting neighbor-distance object then feeds AverageSimilarityPlot, HighSimilarityNonNeighborPlot (fixed axes xlim(-4,1.5), ylim(-3.5,2)), and ModalityConcordancePlot, each saved as PDF via ggplot with a bw theme. Correct object structure is the prerequisite for the spatial homogeneity diagnostics that guide integration-strategy choice among the 54 integrative strategies.

## Related tools

- **Seurat** (Provides the R object framework (spatial assay, reductions, metadata) that stImage reads and manipulates) — https://satijalab.org/seurat
- **stImage** (Consumes the Seurat object in PreProcessing and extractNeighborDist and returns results as Seurat-compatible objects/plots) — https://github.com/YuWang-VUMC/stImage
- **R** (Runtime environment in which Seurat objects are loaded and passed to stImage functions)

## Examples

```
library(stImage); obj <- PreProcessing(obj); nd <- extractNeighborDist(obj); AverageSimilarityPlot(nd)
```

## Evaluation signals

- The Seurat object contains an SCT-normalized spatial assay and both gene and image-feature PCA reductions with 20 PCs
- extractNeighborDist returns a per-spot structure pairing similarity to spatial neighbors against randomly chosen non-neighbors
- The three diagnostic PDFs (HER2_DistConsistency.pdf, HER2_SimilarToNeighbors.pdf, HER2_SimilarToNeighbors2.pdf) are generated and render ggplot figures with the expected titles/layers
- The ModalityConcordancePlot reports the fraction of spots whose image similarity to neighbors exceeds their gene similarity (neighbor-preference), comparable to reported values
- Fixed axis limits on HighSimilarityNonNeighborPlot (xlim -4 to 1.5, ylim -3.5 to 2) produce points within the expected range

## Limitations

- Effectiveness depends on assumptions (spatial homogeneity, strong spatial-functional relationships, image quality/informativeness) that are not always met, so the object-level diagnostics must be checked before choosing an integration strategy
- Multi-modal integration on large objects is computationally more intensive and algorithm-dependent, though stImage runtime remains manageable on a typical office desktop
- Dependencies (SPARK/SPARK-X, SpatialPCA, BayesSpace, tensorflow/keras, reticulate virtualenv) must be installed before the Seurat/stImage objects can be manipulated end-to-end

## Evidence

- [readme] StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images.: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images."
- [readme] Once Tensorflow and keras were successfully installed, you can then install the latest version of stImage from GitHub with:: [evidence span withheld — anchor status: external_doc]
- [abstract] we introduce stImage, an open-source R package that provides a comprehensive and flexible solution for ST analysis: "we introduce stImage, an open-source R package that provides a comprehensive and flexible solution for ST analysis"
- [methods] HER2 breast cancer (Spatial Transcriptomics platform) dataset can be accessed at https://doi.org/10.5281/zenodo.4751624: "HER2 breast cancer (Spatial Transcriptomics platform) dataset can be accessed at https://doi.org/10.5281/zenodo.4751624"
- [readme] Before installing stImage, dependencies should be installed first: [evidence span withheld — anchor status: external_doc]
