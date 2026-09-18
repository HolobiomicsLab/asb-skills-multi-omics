---
name: spatial-marker-gene-visualization
description: Use when validating that spatial clusters from Visium/stImage analysis
  correspond to known anatomical structures by checking marker gene expression (e.g.
  Podxl, Mgp) overlaid on tissue coordinates.
when_to_use_negative:
- When the goal is purely quantitative marker scoring (e.g. differential expression
  testing) rather than spatial validation of cluster anatomy
- When the spatial transcriptomics data has no accompanying image or spatial coordinate
  information to anchor the feature plot
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: stImage
  role: Provides the stImage object and plotting wrapper (AllSpatialFeaturePlot/SpatialFeaturePlot)
    that renders marker expression on tissue space from the Visium input
  repo: https://github.com/YuWang-VUMC/stImage
- name: Seurat
  role: Underlying R toolkit whose SpatialFeaturePlot / SpatialDimPlot functions are
    invoked to generate the spatial visualizations
  repo: https://github.com/satijalab/seurat
- name: tidyverse
  role: Data-manipulation and plotting dependencies used in the R environment for
    preparing the marker/cluster data frames
- name: patchwork
  role: Combines the individual spatial feature/dim plots into a multi-panel figure
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:6ffdd0677a39c7ae6c73efe685cc529624685e7674c042703e0d88b3767f42d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuWang-VUMC/stImage
  tools:
  - R
  - Seurat
  - patchwork
  - stImage
  - tidyverse
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.1111111111111111
n_claims: 9
---

# Spatial marker gene visualization

## Summary

Visualizes cell-type marker gene expression directly on spatial tissue coordinates with stImage/Seurat plotting functions (SpatialFeaturePlot/AllSpatialFeaturePlot), overlaying feature-plot signal against spatial cluster assignments to validate that identified clusters correspond to anatomical structures such as glomeruli or arterial spots.

## When to use

Use when validating that spatial clusters from Visium/stImage analysis correspond to known anatomical structures by checking marker gene expression (e.g. Podxl, Mgp) overlaid on tissue coordinates.

## When NOT to use

- When the goal is purely quantitative marker scoring (e.g. differential expression testing) rather than spatial validation of cluster anatomy
- When the spatial transcriptomics data has no accompanying image or spatial coordinate information to anchor the feature plot

## Inputs

- Seurat/stImage spatial object with normalized expression matrix and spot spatial coordinates (e.g. from Visium filtered_feature_bc_matrix.h5)
- Marker gene list per expected cell type or structure (e.g. Podxl, Synpo, Nphs1, Ehd3 for glomeruli; Mgp, Flna, Tagln, Eln for arteries)
- Cluster assignment column from the chosen integration strategy (e.g. 'wnn_clusterN')

## Outputs

- Spatial feature plots showing per-spot marker expression overlaid on tissue image
- Spatial dim plots showing cluster assignments on tissue space
- Combined figure panels for publication

## How to apply

After clustering (e.g. WNN-integrated clustering from stImage's MultiModalIntegration), project marker gene expression onto spot coordinates using AllSpatialFeaturePlot or SpatialFeaturePlot with cluster.frame = TRUE. For mouse kidney Visium data, use glomerulus markers Podxl, Synpo, Nphs1, Ehd3 and arterial markers Mgp, Flna, Tagln, Eln. Overlay these feature plots alongside SpatialDimPlot of cluster assignments (e.g. group.by = 'wnn_clusterN') to confirm that clusters with high marker signal are spatially co-localized with the expected anatomical substructure. Correct application is judged by whether the feature-plot signal for each marker matches the spatial extent of the corresponding cluster.

## Related tools

- **stImage** (Provides the stImage object and plotting wrapper (AllSpatialFeaturePlot/SpatialFeaturePlot) that renders marker expression on tissue space from the Visium input) — https://github.com/YuWang-VUMC/stImage
- **Seurat** (Underlying R toolkit whose SpatialFeaturePlot / SpatialDimPlot functions are invoked to generate the spatial visualizations) — https://github.com/satijalab/seurat
- **tidyverse** (Data-manipulation and plotting dependencies used in the R environment for preparing the marker/cluster data frames)
- **patchwork** (Combines the individual spatial feature/dim plots into a multi-panel figure)

## Examples

```
SpatialFeaturePlot(stImage_obj, features = c('Podxl','Synpo','Nphs1','Ehd3'), group.by = 'wnn_clusterN', cluster.frame = TRUE)
```

## Evaluation signals

- The WNN spatial cluster plot shows a distinct glomerulus cluster co-localized with Podxl/Synpo/Nphs1/Ehd3 feature-plot signal, matching the paper's Figure 6.
- Arterial clusters identified by clustering are co-localized with Mgp/Flna/Tagln/Eln feature-plot signal on tissue space.
- Feature plots are generated with cluster.frame = TRUE so the color scale is interpretable across panels.
- The cluster-assignment column passed to group.by matches the integration strategy actually used (e.g. wnn_clusterN for WNN results).

## Limitations

- Validation relies on manual inspection of co-localization between cluster boundaries and marker signal; no automated metric is prescribed by the paper.
- The spatial feature plot reflects the quality of the underlying expression matrix and the chosen integration strategy; a poor clustering result (e.g. from a non-WNN strategy) will not recover the glomerulus cluster even though the plotting step is correct.

## Evidence

- [methods] Visualize the SCT, ImageFeature and WNN cluster assignments on tissue space with SpatialDimPlot (group.by = 'wnn_clusterN' for the integrated result).: [evidence span withheld — anchor status: not_found]
- [methods] Plot kidney markers on tissue space with AllSpatialFeaturePlot/SpatialFeaturePlot (cluster.frame = TRUE): Podxl, Synpo, Nphs1, Ehd3 for glomeruli and Mgp, Flna, Tagln, Eln for arterial spots.: [evidence span withheld — anchor status: not_found]
- [results] acceptance check is that the WNN spatial cluster plot shows a distinct glomerulus cluster co-localized with Podxl/Synpo/Nphs1/Ehd3 feature-plot signal: [evidence span withheld — anchor status: not_found]
