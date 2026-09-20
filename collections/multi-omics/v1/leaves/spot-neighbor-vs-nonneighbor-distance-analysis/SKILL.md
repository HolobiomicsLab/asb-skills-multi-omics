---
name: spot-neighbor-vs-nonneighbor-distance-analysis
description: Use when preparing HER2/Visium spatial transcriptomics data for spatial-aware
  methods (BayesSpace, SpatialPCA) and you must verify each spot is closer to spatial
  neighbors than non-neighbors before choosing a strategy.
when_to_use_negative:
- The tissue is spatially uniform and the histological image is uninformative or conflicts
  with gene expression — the article recommends skipping modality diagnostics in favor
  of a spatially aware process alone
- Neither image features nor spatial structure are informative — rely solely on gene
  expression with standard methods such as PCA instead of running neighbor-vs-non-neighbor
  diagnostics
- The input is raw counts without stImage PreProcessing (SCT normalization and 20-PC
  gene/image reductions); extractNeighborDist expects the preprocessed object
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: stImage
  role: Provides extractNeighborDist, AverageSimilarityPlot, HighSimilarityNonNeighborPlot
    and ModalityConcordancePlot used to compute and plot per-spot neighbor vs non-neighbor
    similarities
  repo: https://github.com/YuWang-VUMC/stImage
- name: Seurat
  role: Hosts the preprocessed spatial object (SCT assay, PCA reductions) on which
    the stImage diagnostic functions are run
  repo: https://satijalab.org/seurat
- name: tidyverse
  role: R ecosystem (ggplot2 bw theme, piping) used to render the diagnostic plots
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:111fa998f7ec1004106d4a86c317dd274a89fa0d5ce46c28c43559b4a6a9da36
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuWang-VUMC/stImage
  tools:
  - R
  - Seurat
  - stImage
  - tidyverse
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# spot-neighbor-vs-nonneighbor-distance-analysis

## Summary

A stImage diagnostic workflow that quantifies, per spot, whether each spot is more similar to its spatial neighbors than to randomly chosen non-neighbors in the top-PC embedding space, checking the spatial homogeneity assumption and rendering three ggplot PDFs (Average Similarity, High-Similarity Non-Neighbor, Modality Concordance).

## When to use

Use when preparing HER2/Visium spatial transcriptomics data for spatial-aware methods (BayesSpace, SpatialPCA) and you must verify each spot is closer to spatial neighbors than non-neighbors before choosing a strategy.

## When NOT to use

- The tissue is spatially uniform and the histological image is uninformative or conflicts with gene expression — the article recommends skipping modality diagnostics in favor of a spatially aware process alone
- Neither image features nor spatial structure are informative — rely solely on gene expression with standard methods such as PCA instead of running neighbor-vs-non-neighbor diagnostics
- The input is raw counts without stImage PreProcessing (SCT normalization and 20-PC gene/image reductions); extractNeighborDist expects the preprocessed object

## Inputs

- Preprocessed Seurat spatial object with SCT-normalized spatial assay, gene PCA and image-feature PCA reductions (20 PCs), e.g. HER2 breast cancer ST data from Zenodo 10.5281/zenodo.4751624
- Spot spatial coordinates / neighbor graph implied by the Seurat spatial assay
- Image-feature embeddings from stImage LoadImageFeature / PreProcessing

## Outputs

- HER2_DistConsistency.pdf — paired gene-vs-image neighbor-similarity concordance ggplot titled 'HER2'
- HER2_SimilarToNeighbors.pdf — spot distance-to-neighbors ggplot from AverageSimilarityPlot
- HER2_SimilarToNeighbors2.pdf — high-similarity non-neighbor ggplot with fixed axes xlim(-4,1.5), ylim(-3.5,2)
- Per-spot neighbor vs non-neighbor similarity/distance table from extractNeighborDist

## How to apply

Load the preprocessed HER2 Seurat object (SCT-normalized spatial assay with gene and image-feature PCA reductions, 20 PCs) produced by the stImage PreProcessing step. Run stImage's extractNeighborDist on the object so that, for every spot, its similarity/distance to spatial neighbors versus randomly chosen non-neighbors is computed in the top-PC embedding space. Call ModalityConcordancePlot to build the paired gene-vs-image similarity scatter (saved as HER2_DistConsistency.pdf, titled 'HER2', bw theme base_size 16, no legend) and AverageSimilarityPlot on the neighbor-distance result to produce HER2_SimilarToNeighbors.pdf. Call HighSimilarityNonNeighborPlot with fixed axes xlim(-4,1.5) and ylim(-3.5,2) to produce HER2_SimilarToNeighbors2.pdf. Use the concordance plot's reported fraction of spots whose image similarity to neighbors exceeds their gene similarity (neighbor-preference) to decide whether a spatial-aware process, modality integration, or gene-only PCA is appropriate.

## Related tools

- **stImage** (Provides extractNeighborDist, AverageSimilarityPlot, HighSimilarityNonNeighborPlot and ModalityConcordancePlot used to compute and plot per-spot neighbor vs non-neighbor similarities) — https://github.com/YuWang-VUMC/stImage
- **Seurat** (Hosts the preprocessed spatial object (SCT assay, PCA reductions) on which the stImage diagnostic functions are run) — https://satijalab.org/seurat
- **tidyverse** (R ecosystem (ggplot2 bw theme, piping) used to render the diagnostic plots)

## Examples

```
dist_res <- extractNeighborDist(obj); ModalityConcordancePlot(obj); AverageSimilarityPlot(dist_res); HighSimilarityNonNeighborPlot(dist_res)
```

## Evaluation signals

- All three PDFs (HER2_DistConsistency.pdf, HER2_SimilarToNeighbors.pdf, HER2_SimilarToNeighbors2.pdf) are generated and each renders a ggplot figure with the expected titles/layers
- The concordance plot reports the fraction of spots whose image similarity to neighbors exceeds their gene similarity (neighbor-preference), comparable against reported values
- HighSimilarityNonNeighborPlot output uses the fixed axes xlim(-4,1.5) and ylim(-3.5,2)
- extractNeighborDist returns per-spot neighbor vs non-neighbor distances for every spot in the object (no missing spots)
- ModalityConcordancePlot PDF is titled 'HER2', uses the bw theme with base_size 16, and has no legend

## Limitations

- The diagnostics check assumptions (spatial homogeneity, strong spatial-functional relationships, image quality) that are not always met, so results do not guarantee a universal integration strategy — no single approach performed well on every dataset tested
- Multi-modal integration is computationally more intensive and algorithm-dependent, though overall runtime remains manageable on a typical office desktop
- Diagnostic content for the four decision scenarios (Supplementary Fig. S13 / Table S1) is referenced but not provided in the article text

## Evidence

- [abstract] We demonstrate stImage's effectiveness across multiple datasets, underscoring its ability to guide users toward the most suitable integration strategy using diagnostic graph.: "We demonstrate stImage’s effectiveness across multiple datasets, underscoring its ability to guide users toward the most suitable integration strategy using diagnostic graph."
- [intro] stImage enables users to apply and compare different methods simultaneously, as well as make data-driven decisions using its diagnostic graphs that reflect the spatial and histological: "stImage enables users to apply and compare different methods simultaneously, as well as make data-driven decisions using its diagnostic graphs that reflect the spatial and histological"
- [intro] BASS and BayesSpace leverage spatial neighborhood information, motivating the check of the spatial homogeneity assumption per spot.: "BASS [16] and BayesSpace [14] leverage spatial neighborhood information through Bayesian statistical strategies"
- [readme] StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images.: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images."
