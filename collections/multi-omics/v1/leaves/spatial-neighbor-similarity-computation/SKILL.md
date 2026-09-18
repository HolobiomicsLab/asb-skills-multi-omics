---
name: spatial-neighbor-similarity-computation
description: Use when you need to check whether each spot in a spatial transcriptomics
  dataset is more similar to its spatial neighbors than to non-neighbors before applying
  spatial-aware analysis or choosing an integration strategy.
when_to_use_negative:
- When neither image features nor spatial structure are informative and you plan to
  rely solely on gene expression with standard PCA — the diagnostic is unnecessary
- When the input object has not been through stImage PreProcessing (no SCT-normalized
  assay or 20-PC gene/image reductions), since extractNeighborDist operates in the
  top-PC embedding space
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3608
tools:
- name: stImage
  role: Provides extractNeighborDist, ModalityConcordancePlot, AverageSimilarityPlot,
    and HighSimilarityNonNeighborPlot for neighbor-similarity computation and diagnostic
    plotting
  repo: https://github.com/YuWang-VUMC/stImage
- name: Seurat
  role: Underlying object framework and PreProcessing pipeline (SCT normalization,
    PCA reductions) supplying the embedding space used by extractNeighborDist
  repo: https://github.com/satijalab/seurat
- name: tidyverse
  role: R package stack used in the stImage methods/vignettes supporting ggplot-based
    diagnostic figure rendering (bw theme)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:4cab797946899c649b684270dffffc9ce83e6cd91de03a4451b8e77f8b70c8ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuWang-VUMC/stImage
  tools:
  - R
  - Seurat
  - stImage
  - tidyverse
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3608
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# spatial-neighbor-similarity-computation

## Summary

Computation of per-spot similarity/distance to spatial neighbors versus non-neighbors in a top-PC embedding space (stImage extractNeighborDist), used to assess the spatial homogeneity assumption and produce three diagnostic plots (Average Similarity, High-Similarity Non-Neighbor, Modality Concordance).

## When to use

Use when you need to check whether each spot in a spatial transcriptomics dataset is more similar to its spatial neighbors than to non-neighbors before applying spatial-aware analysis or choosing an integration strategy.

## When NOT to use

- When neither image features nor spatial structure are informative and you plan to rely solely on gene expression with standard PCA — the diagnostic is unnecessary
- When the input object has not been through stImage PreProcessing (no SCT-normalized assay or 20-PC gene/image reductions), since extractNeighborDist operates in the top-PC embedding space

## Inputs

- Preprocessed Seurat spatial transcriptomics object (e.g., HER2 breast cancer ST data) with SCT-normalized spatial assay, gene PCA and image-feature PCA reductions (20 PCs)
- Spatial coordinates/neighbor adjacency derived from spot locations
- Deep learning-derived histology image features (from stImage LoadImageFeature)

## Outputs

- Neighbor-distance result object (per-spot neighbor vs non-neighbor similarities/distances in top-PC space)
- HER2_DistConsistency.pdf — modality concordance scatter (gene vs image neighbor similarity) with neighbor-preference fraction
- HER2_SimilarToNeighbors.pdf — Average Similarity plot (spot distance to neighbors)
- HER2_SimilarToNeighbors2.pdf — High-Similarity Non-Neighbor plot (fixed axes xlim(-4,1.5), ylim(-3.5,2))

## How to apply

Load the preprocessed object (e.g., the HER2 breast cancer ST dataset with SCT-normalized spatial assay, gene and image-feature PCA reductions at 20 PCs) produced by stImage's PreProcessing step. Run stImage's extractNeighborDist on the object so that, for every spot, its similarity/distance to spatial neighbors is compared against randomly chosen non-neighbors in the top-PC embedding space. Pass the resulting neighbor-distance object to three diagnostic plotting functions: ModalityConcordancePlot (paired gene-vs-image similarity scatter, saved as HER2_DistConsistency.pdf titled 'HER2' with bw theme, base size 16, no legend), AverageSimilarityPlot (spot-distance-to-neighbors plot, saved as HER2_SimilarToNeighbors.pdf), and HighSimilarityNonNeighborPlot (similarity-to-neighbors plot with fixed axes xlim(-4,1.5) and ylim(-3.5,2), saved as HER2_SimilarToNeighbors2.pdf). The rationale is that spatial-aware methods (GraphST, SpaGCN, SpatialPCA, BASS, BayesSpace) assume spots resemble their neighbors, and these diagnostics quantify per-spot whether that assumption holds; the concordance plot additionally reports the fraction of spots whose image similarity to neighbors exceeds their gene similarity.

## Related tools

- **stImage** (Provides extractNeighborDist, ModalityConcordancePlot, AverageSimilarityPlot, and HighSimilarityNonNeighborPlot for neighbor-similarity computation and diagnostic plotting) — https://github.com/YuWang-VUMC/stImage
- **Seurat** (Underlying object framework and PreProcessing pipeline (SCT normalization, PCA reductions) supplying the embedding space used by extractNeighborDist) — https://github.com/satijalab/seurat
- **tidyverse** (R package stack used in the stImage methods/vignettes supporting ggplot-based diagnostic figure rendering (bw theme))

## Examples

```
nd <- extractNeighborDist(obj); AverageSimilarityPlot(nd, save.path='HER2_SimilarToNeighbors.pdf'); HighSimilarityNonNeighborPlot(nd, save.path='HER2_SimilarToNeighbors2.pdf')
```

## Evaluation signals

- All three PDFs (HER2_DistConsistency.pdf, HER2_SimilarToNeighbors.pdf, HER2_SimilarToNeighbors2.pdf) are generated and render ggplot figures with expected titles/layers
- High-Similarity Non-Neighbor plot uses the fixed axes xlim(-4,1.5) and ylim(-3.5,2)
- Concordance plot reports the fraction of spots whose image similarity to neighbors exceeds their gene similarity (neighbor-preference), comparable against reported values
- extractNeighborDist returns per-spot mean similarity to spatial neighbors vs randomly chosen non-neighbors for every spot
- The concordance figure uses the bw theme with base size 16 and no legend, titled 'HER2'

## Limitations

- The diagnostics depend on assumptions about transcriptomics, spatial, and image patterns that are not always met; they cannot be interpreted when spot homogeneity or informative histology fails entirely
- Diagnostic conclusions are dataset-dependent — no single integration approach performed well on every dataset tested
- Multi-modal integration steps built on these diagnostics are computationally more intensive and algorithm-dependent, though stImage runtime remains manageable on a typical office desktop

## Evidence

- [results] The Average Similarity Plot diagnostic computes, for each spot, its mean similarity to its spatial neighbors versus a set of randomly chosen non-neighbors: [evidence span withheld — anchor status: not_found]
- [methods] Run stImage's extractNeighborDist on the object to compute, for every spot, its similarity/distance to spatial neighbors versus non-neighbors in the top-PC embedding space.: [evidence span withheld — anchor status: not_found]
- [intro] GraphST [15] and SpaGCN [7] utilize graph convolutional networks to derive spot representations by minimizing the embedding distances between spatially adjacent spots: "GraphST [15] and SpaGCN [7] utilize graph convolutional networks to derive spot representations by minimizing the embedding distances between spatially adjacent spots"
- [readme] stImage is freely available at https://github.com/YuWang-VUMC/stImage.: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images. StImage is freely available at https://github.com/YuWang-VUMC/stImage."
