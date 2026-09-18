---
name: spot-level-feature-plotting
description: Use when you have an stImage-processed spatial object with SCT and/or
  WNN-integrated reductions and need spatial feature maps or cluster-stratified expression
  summaries of marker genes (e.g. ERBB2, TIMP1, CFD, CD74) to validate clustering
  against pathologist annotations.
when_to_use_negative:
- When the spatially aware process should be skipped and only modality integration
  applied, or vice versa — the plotting step presupposes clusters already computed
  via stImage's PreProcessing/FindSNNClusters/MultiModalIntegration pipeline
- When neither image features nor spatial structure are informative and only gene
  expression with standard PCA is warranted; there are no WNN clusters to highlight
- When spots have not been preprocessed (e.g. the 21 ring pattern genes from the original
  paper not yet removed), since raw artifacts can distort apparent marker spatial
  patterns
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: stImage
  role: Provides AllSpatialFeaturePlot for SCT-normalized, WNN-integrated spot-level
    feature plotting with cluster highlighting
  repo: https://github.com/YuWang-VUMC/stImage
- name: Seurat
  role: Underlying R object framework (assays, reductions, spatial coordinates) on
    which stImage plotting operates
  repo: https://satijalab.org/seurat
- name: tidyverse
  role: Used in the stImage workflow for data manipulation and figure assembly of
    the marker panels
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:ff039449621034d611c10e1114d5f4e6045dd848aabadf5ac3cd16abdb2b638e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuWang-VUMC/stImage
  tools:
  - R
  - Seurat
  - SpatialPCA
  - stImage
  - tidyverse
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 12
---

# spot-level-feature-plotting

## Summary

Visualization of spot-level gene expression over spatial transcriptomics tissue sections using stImage's AllSpatialFeaturePlot, supporting SCT-normalized or WNN-integrated values with per-cluster highlighting, to validate that clusters recapitulate region-specific marker expression (e.g. ERBB2 in the HER2 H1 in-situ cancer region).

## When to use

Use when you have an stImage-processed spatial object with SCT and/or WNN-integrated reductions and need spatial feature maps or cluster-stratified expression summaries of marker genes (e.g. ERBB2, TIMP1, CFD, CD74) to validate clustering against pathologist annotations.

## When NOT to use

- When the spatially aware process should be skipped and only modality integration applied, or vice versa — the plotting step presupposes clusters already computed via stImage's PreProcessing/FindSNNClusters/MultiModalIntegration pipeline
- When neither image features nor spatial structure are informative and only gene expression with standard PCA is warranted; there are no WNN clusters to highlight
- When spots have not been preprocessed (e.g. the 21 ring pattern genes from the original paper not yet removed), since raw artifacts can distort apparent marker spatial patterns

## Inputs

- stImage object with 'Spatial' assay, SCT-normalized expression, SpatialPCA+SCT and ImageFeature reductions, and WNN integration clusters (e.g. HER2 H1, clusterNum = 6)
- List of marker genes with expected region assignments (e.g. ERBB2, TIMP1, CFD, CD74)
- Pathologist-annotated region labels for the tissue section (for validation)

## Outputs

- Per-marker spatial feature plots over spot coordinates with cluster framing/highlighting (six-color palette)
- Mean marker expression per WNN cluster (region-specificity check)
- Composite figure panel matching the Fig. 3C marker-validation layout

## How to apply

Load the stImage object (e.g. HER2 H1 with SpatialPCA+SCT and ImageFeature reductions and WNN clusters, clusterNum = 6), set the default assay to 'Spatial', and extract SCT-normalized expression for each marker at every spot. For each marker, call stImage's AllSpatialFeaturePlot with normalizeMethod='SCT' and integreation.method='wnn'; set cluster.frame=TRUE and cluster.highlight to the cluster you expect the marker to define (e.g. cluster 3 for ERBB2), using the custom six-color palette for the six WNN clusters. Then compare mean marker expression across WNN clusters to verify region specificity: ERBB2 exclusive to the in-situ cancer cluster, TIMP1 enriched in the invasive cancer cluster, CFD in adipose, and CD74 in the immune infiltrate. Finally, assemble the per-marker spatial plots into a single multi-panel figure (as in the paper's Fig. 3C marker-validation layout).

## Related tools

- **stImage** (Provides AllSpatialFeaturePlot for SCT-normalized, WNN-integrated spot-level feature plotting with cluster highlighting) — https://github.com/YuWang-VUMC/stImage
- **Seurat** (Underlying R object framework (assays, reductions, spatial coordinates) on which stImage plotting operates) — https://satijalab.org/seurat
- **tidyverse** (Used in the stImage workflow for data manipulation and figure assembly of the marker panels)

## Examples

```
AllSpatialFeaturePlot(stImage_obj, features = 'ERBB2', normalizeMethod = 'SCT', integreation.method = 'wnn', cluster.frame = TRUE, cluster.highlight = 3)
```

## Evaluation signals

- ERBB2 expression is exclusive to WNN cluster 3, corresponding to the pathologist-annotated in-situ cancer region (reproducing Fig. 3C)
- TIMP1 is enriched in the invasive cancer cluster, CFD in the adipose cluster, and CD74 in the immune infiltrate cluster
- Feature plots are generated with normalizeMethod='SCT' and integreation.method='wnn', cluster.frame=TRUE, and the custom six-color palette consistent with clusterNum = 6
- All four marker plots assemble into one figure panel matching the Fig. 3C layout

## Limitations

- The article identifies a gap that existing methods focus on gene expression with image playing a secondary role; interpretation of marker plots should account for the chosen integration strategy, as no single approach performed well on every dataset
- Multi-modal integration is computationally more intensive and algorithm-dependent, which affects the reductions feeding the feature plots, though stImage runtime remains manageable on a typical office desktop
- Only the four documented markers (ERBB2, TIMP1, CFD, CD74) have validated region-specific patterns in the HER2 H1 dataset; other markers lack a ground-truth cluster assignment

## Evidence

- [abstract] stImage combines transcriptional profiles, histology images, and spatial information: "stImage seamlessly combines transcriptional profiles, histology images, and spatial information"
- [methods] 21 ring pattern gene removal: "remove the 21 ring pattern genes mentioned in the orginal paper"
- [readme] stImage package definition: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images."
- [discussion] no single best integration strategy: "As we expected, there was no single approach performing well in every dataset."
