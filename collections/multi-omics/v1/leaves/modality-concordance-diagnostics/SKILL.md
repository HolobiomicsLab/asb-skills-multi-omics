---
name: modality-concordance-diagnostics
description: Use when preparing a spatial transcriptomics dataset (e.g. HER2 ST or
  Visium) for spatial-aware or multi-modal analysis and you must decide whether spots
  are more similar to spatial neighbors than non-neighbors before choosing an integration
  strategy.
when_to_use_negative:
- The dataset is not spatially resolved (no spot coordinates or neighbor structure),
  since neighbor vs non-neighbor comparisons are undefined
- Image features have not yet been extracted/loaded (LoadImageFeature not run), so
  the modality concordance scatter cannot be computed
- You only need downstream clustering or integration (FindSNNClusters, MultiModalIntegration)
  and have already accepted the spatial homogeneity assumption from a prior diagnostic
  run
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: stImage
  role: Provides extractNeighborDist, AverageSimilarityPlot, HighSimilarityNonNeighborPlot,
    and ModalityConcordancePlot functions that generate the three diagnostic plots
  repo: https://github.com/YuWang-VUMC/stImage
- name: Seurat
  role: Supplies the spatial object class, SCT normalization, and PCA reductions that
    extractNeighborDist operates on
  repo: https://satijalab.org/seurat
- name: tidyverse
  role: Used in the R session for the ggplot-based diagnostic figure rendering (bw
    theme, axis limits)
- name: R
  role: Execution environment for the stImage diagnostic workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:0a3839072e75f627971ab6ff47c6b0f78c32bbba6035239ef6dd424256cd73bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuWang-VUMC/stImage
  tools:
  - R
  - Seurat
  - stImage
  - tidyverse
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# modality-concordance-diagnostics

## Summary

Reconstructs stImage's three diagnostic plots (Average Similarity, High-Similarity Non-Neighbor, Modality Concordance) that quantify whether individual spots satisfy the spatial homogeneity assumption and whether image features align with gene expression, guiding the choice among stImage's 54 integrative strategies.

## When to use

Use when preparing a spatial transcriptomics dataset (e.g. HER2 ST or Visium) for spatial-aware or multi-modal analysis and you must decide whether spots are more similar to spatial neighbors than non-neighbors before choosing an integration strategy.

## When NOT to use

- The dataset is not spatially resolved (no spot coordinates or neighbor structure), since neighbor vs non-neighbor comparisons are undefined
- Image features have not yet been extracted/loaded (LoadImageFeature not run), so the modality concordance scatter cannot be computed
- You only need downstream clustering or integration (FindSNNClusters, MultiModalIntegration) and have already accepted the spatial homogeneity assumption from a prior diagnostic run

## Inputs

- Preprocessed Seurat spatial object from stImage PreProcessing (SCT-normalized spatial assay with gene PCA and image-feature PCA reductions, 20 PCs)
- Spatial neighbor adjacency derived from spot coordinates on the ST/Visium platform
- Deep learning-derived histology image features loaded via LoadImageFeature

## Outputs

- HER2_DistConsistency.pdf — gene-vs-image similarity concordance scatter with fraction of neighbor-preferred spots
- HER2_SimilarToNeighbors.pdf — Average Similarity plot of per-spot distance to neighbors
- HER2_SimilarToNeighbors2.pdf — High-Similarity Non-Neighbor plot with fixed axes xlim(-4,1.5), ylim(-3.5,2)
- extractNeighborDist result object with per-spot neighbor vs non-neighbor distances/similarities

## How to apply

Load the preprocessed object produced by the stImage PreProcessing step (SCT-normalized spatial assay with gene and image-feature PCA reductions, 20 PCs). Run stImage's extractNeighborDist on the object to compute, for every spot, its similarity/distance to spatial neighbors versus randomly chosen non-neighbors in the top-PC embedding space. Then generate three ggplot PDFs: AverageSimilarityPlot (HER2_SimilarToNeighbors.pdf), HighSimilarityNonNeighborPlot with fixed axes xlim(-4,1.5) and ylim(-3.5,2) (HER2_SimilarToNeighbors2.pdf), and ModalityConcordancePlot for the paired gene-vs-image similarity scatter (HER2_DistConsistency.pdf, titled 'HER2', bw theme base size 16, no legend). The concordance plot reports the fraction of spots whose image similarity to neighbors exceeds their gene similarity (neighbor-preference); use this fraction plus the distance plots to decide whether to apply a spatial-aware process, modality integration, both, or neither.

## Related tools

- **stImage** (Provides extractNeighborDist, AverageSimilarityPlot, HighSimilarityNonNeighborPlot, and ModalityConcordancePlot functions that generate the three diagnostic plots) — https://github.com/YuWang-VUMC/stImage
- **Seurat** (Supplies the spatial object class, SCT normalization, and PCA reductions that extractNeighborDist operates on) — https://satijalab.org/seurat
- **tidyverse** (Used in the R session for the ggplot-based diagnostic figure rendering (bw theme, axis limits))
- **R** (Execution environment for the stImage diagnostic workflow)

## Examples

```
library(stImage); nd <- extractNeighborDist(obj); ModalityConcordancePlot(obj, title='HER2', file='HER2_DistConsistency.pdf'); AverageSimilarityPlot(nd, file='HER2_SimilarToNeighbors.pdf'); HighSimilarityNonNeighborPlot(nd, xlim=c(-4,1.5), ylim=c(-3.5,2), file='HER2_SimilarToNeighbors2.pdf')
```

## Evaluation signals

- All three PDFs (HER2_DistConsistency.pdf, HER2_SimilarToNeighbors.pdf, HER2_SimilarToNeighbors2.pdf) are generated and render ggplot figures with expected titles/layers
- ModalityConcordancePlot output reports a fraction of spots whose image similarity to neighbors exceeds gene similarity, comparable against values reported in the paper
- High-Similarity Non-Neighbor plot uses the fixed axis ranges xlim(-4,1.5) and ylim(-3.5,2)
- Concordance figure uses theme_bw with base size 16, title 'HER2', and no legend
- extractNeighborDist returns distances for every spot in the top-PC embedding space (neighbors vs random non-neighbors) without missing spots

## Limitations

- The diagnostics test assumptions (spatial homogeneity, image informativeness) that are 'not always met, rendering a universal, one-size-fits-all solution elusive' — a low concordance fraction means spatial-aware processing should be skipped
- No single integration approach performed well on every dataset tested, so the diagnostic outcome must be interpreted per dataset rather than treated as a universal rule
- Multi-modal integration is computationally more intensive and algorithm-dependent, though stImage runtime remains manageable on a typical office desktop
- Supplementary Fig. S13 and Table S1 detailing the four decision scenarios are referenced but not provided, limiting full reproducibility of the decision thresholds

## Evidence

- [methods] The Average Similarity Plot diagnostic computes, for each spot, its mean similarity to its spatial neighbors versus a set of randomly chosen non-neighbors: [evidence span withheld — anchor status: not_found]
- [methods] Call ModalityConcordancePlot on the object to build the paired gene-vs-image similarity scatter and save it as HER2_DistConsistency.pdf titled 'HER2' with a bw theme (base size 16) and no legend: [evidence span withheld — anchor status: not_found]
- [intro] stImage enables users to apply and compare different methods simultaneously, as well as make data-driven decisions using its diagnostic graphs that reflect the spatial and histological: "stImage enables users to apply and compare different methods simultaneously, as well as make data-driven decisions using its diagnostic graphs that reflect the spatial and histological"
- [methods] Load the preprocessed HER2 object (SCT-normalized spatial assay with gene and image-feature PCA reductions, 20 PCs) produced by the stImage PreProcessing step: [evidence span withheld — anchor status: not_found]
- [readme] StImage is freely available at https://github.com/YuWang-VUMC/stImage.: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images. StImage is freely available at https://github.com/YuWang-VUMC/stImage."
