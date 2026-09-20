---
name: ggplot-spatial-diagnostics-visualization
description: Use when you have a preprocessed Seurat spatial object (e.g., SCT-normalized
  HER2 or MOB assay with gene and image-feature PCA reductions) and must diagnose
  whether spots are more similar to spatial neighbors than non-neighbors before choosing
  an stImage strategy.
when_to_use_negative:
- You only need a single statistical metric and no per-spot diagnostic figure — the
  fraction of neighbor-preference spots is already reported and does not require the
  plotting step
- The tissue is spatially uniform and the histological image is uninformative or conflicts
  with gene expression, so the diagnostic's recommendation is already to skip modality
  integration and use only a spatially aware process
- Input lacks spatial coordinates and histology-derived image features, since extractNeighborDist
  and the concordance plot both require the spatial and image modalities
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0085
tools:
- name: stImage
  role: Provides extractNeighborDist, AverageSimilarityPlot, HighSimilarityNonNeighborPlot,
    and ModalityConcordancePlot functions that generate the diagnostic ggplot figures
  repo: https://github.com/YuWang-VUMC/stImage
- name: R
  role: Runtime language in which the stImage diagnostic plotting functions are executed
- name: tidyverse
  role: Loaded in the methods workflow and provides ggplot2 styling (theme_bw, bw
    theme) used by the diagnostic plots
- name: Seurat
  role: Provides the preprocessed spatial object structure (SCT-normalized assay,
    PCA reductions) that the diagnostic functions consume
  repo: satijalab/seurat
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:3c46731546b2fa29dd8c21b14e23c0cefb8a63108a5e8f82910b0679189baa66
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# ggplot-spatial-diagnostics-visualization

## Summary

Reproduce stImage's three ggplot-based spatial diagnostic plots — AverageSimilarityPlot (spot distance to neighbors), HighSimilarityNonNeighborPlot (similarity to neighbors with fixed axes), and ModalityConcordancePlot (gene-vs-image similarity concordance) — that quantify whether spots conform to the spatial homogeneity assumption before applying spatial-aware or modality-integration strategies.

## When to use

Use when you have a preprocessed Seurat spatial object (e.g., SCT-normalized HER2 or MOB assay with gene and image-feature PCA reductions) and must diagnose whether spots are more similar to spatial neighbors than non-neighbors before choosing an stImage strategy.

## When NOT to use

- You only need a single statistical metric and no per-spot diagnostic figure — the fraction of neighbor-preference spots is already reported and does not require the plotting step
- The tissue is spatially uniform and the histological image is uninformative or conflicts with gene expression, so the diagnostic's recommendation is already to skip modality integration and use only a spatially aware process
- Input lacks spatial coordinates and histology-derived image features, since extractNeighborDist and the concordance plot both require the spatial and image modalities

## Inputs

- Preprocessed Seurat spatial object from stImage PreProcessing (SCT-normalized spatial assay, gene PCA and image-feature PCA reductions, 20 PCs)
- Neighbor-distance result object from stImage extractNeighborDist
- HER2 breast cancer ST dataset (Zenodo DOI 10.5281/zenodo.4751624) or Adult Mouse Olfactory Bulb Visium dataset

## Outputs

- HER2_SimilarToNeighbors.pdf — ggplot spot-distance-to-neighbors figure from AverageSimilarityPlot
- HER2_SimilarToNeighbors2.pdf — ggplot similarity-to-neighbors figure from HighSimilarityNonNeighborPlot (xlim -4 to 1.5, ylim -3.5 to 2)
- HER2_DistConsistency.pdf — paired gene-vs-image similarity concordance scatter titled 'HER2', theme_bw(base_size = 16), no legend
- Fraction of spots with neighbor-preference (image similarity to neighbors exceeds gene similarity)

## How to apply

First run stImage's extractNeighborDist on the preprocessed object (SCT-normalized spatial assay, gene and image-feature PCA, 20 PCs from the PreProcessing step) to compute, for every spot, similarity/distance to spatial neighbors versus randomly chosen non-neighbors in the top-PC embedding space. Then call ModalityConcordancePlot to build the paired gene-vs-image similarity scatter, saved as HER2_DistConsistency.pdf with title 'HER2', theme_bw(base_size = 16), and no legend; it reports the fraction of spots whose image similarity to neighbors exceeds their gene similarity (neighbor-preference). Call AverageSimilarityPlot on the neighbor-distance result to produce the spot-distance-to-neighbors plot (HER2_SimilarToNeighbors.pdf), and HighSimilarityNonNeighborPlot with fixed axes xlim(-4, 1.5) and ylim(-3.5, 2) for the similarity-to-neighbors plot (HER2_SimilarToNeighbors2.pdf). The rationale is that spatial-aware methods (SpaGCN, SpatialPCA, BayesSpace, BASS) minimize embedding distances between adjacent spots, so if spots resemble non-neighbors more than neighbors, the spatial homogeneity assumption fails and modality integration or gene-only PCA should be preferred instead.

## Related tools

- **stImage** (Provides extractNeighborDist, AverageSimilarityPlot, HighSimilarityNonNeighborPlot, and ModalityConcordancePlot functions that generate the diagnostic ggplot figures) — https://github.com/YuWang-VUMC/stImage
- **R** (Runtime language in which the stImage diagnostic plotting functions are executed)
- **tidyverse** (Loaded in the methods workflow and provides ggplot2 styling (theme_bw, bw theme) used by the diagnostic plots)
- **Seurat** (Provides the preprocessed spatial object structure (SCT-normalized assay, PCA reductions) that the diagnostic functions consume) — satijalab/seurat

## Examples

```
obj <- extractNeighborDist(obj); ModalityConcordancePlot(obj, file = "HER2_DistConsistency.pdf", title = "HER2"); AverageSimilarityPlot(obj, file = "HER2_SimilarToNeighbors.pdf"); HighSimilarityNonNeighborPlot(obj, file = "HER2_SimilarToNeighbors2.pdf")
```

## Evaluation signals

- Each of the three PDFs (HER2_DistConsistency.pdf, HER2_SimilarToNeighbors.pdf, HER2_SimilarToNeighbors2.pdf) is generated and renders a ggplot figure
- The concordance plot carries the expected title 'HER2', a bw theme with base_size 16, and no legend
- HighSimilarityNonNeighborPlot uses the fixed axes xlim(-4, 1.5) and ylim(-3.5, 2)
- The concordance plot reports the fraction of spots whose image similarity to neighbors exceeds their gene similarity (neighbor-preference), which can be compared against reported values
- extractNeighborDist returns per-spot neighbor vs non-neighbor similarities in the top-PC embedding space for every spot

## Limitations

- No single integration approach performed well on every dataset, so the diagnostic graphs must be interpreted per dataset rather than used as a universal rule
- The fraction of neighbor-preference spots depends on the PCA embedding (20 PCs) and the random selection of non-neighbors, so it is not deterministic
- Multi-modal integration is computationally more intensive and algorithm-dependent, though stImage runtime remains manageable on a typical office desktop

## Evidence

- [abstract] We demonstrate stImage’s effectiveness across multiple datasets, underscoring its ability to guide users toward the most suitable integration strategy using diagnostic graph.: "We demonstrate stImage’s effectiveness across multiple datasets, underscoring its ability to guide users toward the most suitable integration strategy using diagnostic graph."
- [methods] Distance Consistency heading: "## Distance Consistency"
- [methods] Spot Distance To Neighbors heading: "## Spot Distance To Neighbors"
- [methods] Similarity To Neighbors heading: "## Similarity To Neighbors"
- [readme] stImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images.: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images."
