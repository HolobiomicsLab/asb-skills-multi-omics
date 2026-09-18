---
name: niche-cell-colocalization-analysis
description: Use when you have an annotated image-based spatial omics dataset (e.g.,
  MERSCOPE) with cell-type labels and niche assignments, and you need to test whether
  specific cell populations co-localize with particular tissue niches.
when_to_use_negative:
- The input data is raw images/transcripts that have not yet been segmented, aggregated,
  and annotated into a SpatialData object with cell-type and niche labels
- The question concerns segmentation quality or gene/transcript filtering (min_area,
  min_transcripts, gene_exclude_pattern) rather than spatial colocalization
- You need physical micron-scale distances rather than graph hop distances, since
  Delaunay hop-distance statistics measure neighbor-graph connectivity
tools:
- name: Sopa
  role: Provides sopa.spatial.spatial_neighbors (Delaunay graph), sopa.spatial.cells_to_groups,
    sopa.spatial.mean_distance, and niches_geometry_stats for the hop-distance colocalization
    analysis
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Unified data framework holding the cell boundaries, counts, and annotations
    on which the spatial analysis is performed
  repo: https://github.com/scverse/spatialdata
- name: MERSCOPE
  role: Image-based spatial transcriptomics technology generating the liver dataset
    used to demonstrate niche-specific macrophage annotation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:7dd636c2c6bdf760226cbf7dccc8a40695b66a2f917efcd0b5a6d008c1c85c79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - MERSCOPE
  - Sopa
  - SpatialData
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# niche-cell-colocalization-analysis

## Summary

A Sopa-based spatial statistics skill that quantifies cell-type/niche colocalization via mean hop distances on a Delaunay cell-neighbor graph, used to detect spatially specific populations such as TREM2, LRP1, and CEBP macrophages confined to the necrotic niche in the MERSCOPE liver dataset.

## When to use

Use when you have an annotated image-based spatial omics dataset (e.g., MERSCOPE) with cell-type labels and niche assignments, and you need to test whether specific cell populations co-localize with particular tissue niches.

## When NOT to use

- The input data is raw images/transcripts that have not yet been segmented, aggregated, and annotated into a SpatialData object with cell-type and niche labels
- The question concerns segmentation quality or gene/transcript filtering (min_area, min_transcripts, gene_exclude_pattern) rather than spatial colocalization
- You need physical micron-scale distances rather than graph hop distances, since Delaunay hop-distance statistics measure neighbor-graph connectivity

## Inputs

- Annotated SpatialData object with cell boundaries polygons, cell-by-gene counts, cell-type labels, and niche assignments (MERSCOPE liver dataset format)
- Cell-type annotation table (e.g., macrophage subpopulations LRP1, CEBP, TREM2)
- Niche assignment labels per cell

## Outputs

- Cell-type x cell-type mean hop-distance matrix
- Macrophage-subset x niche mean hop-distance matrix
- Niche-level geometry summary statistics (niches_geometry_stats)
- Heatmap visualizations of the hop-distance matrices

## How to apply

Load the annotated SpatialData object containing cell boundaries, cell-type labels, and niche assignments. Build the cell-neighbor graph with sopa.spatial.spatial_neighbors using Delaunay graph construction, then link cell types to their niches with sopa.spatial.cells_to_groups. Compute mean hop distances between cell-type and niche pairs with sopa.spatial.mean_distance and summarize niche-level geometry with niches_geometry_stats. Render heatmaps — one cell-type x cell-type and one macrophage-subset x niche — and interpret minimal hop distance to self plus restricted niche co-occurrence (e.g., necrotic niche) as evidence of spatial specificity, using niche-independent populations (conventional DCs) as the counter-example.

## Related tools

- **Sopa** (Provides sopa.spatial.spatial_neighbors (Delaunay graph), sopa.spatial.cells_to_groups, sopa.spatial.mean_distance, and niches_geometry_stats for the hop-distance colocalization analysis) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Unified data framework holding the cell boundaries, counts, and annotations on which the spatial analysis is performed) — https://github.com/scverse/spatialdata
- **MERSCOPE** (Image-based spatial transcriptomics technology generating the liver dataset used to demonstrate niche-specific macrophage annotation)

## Examples

```
import sopa.spatial; adata = sdata.tables['table']; sopa.spatial.spatial_neighbors(adata, coordinates_key='centroids'); sopa.spatial.mean_distance(adata, key='cell_type')
```

## Evaluation signals

- LRP1, CEBP, and TREM2 macrophages show minimal mean hop distance to themselves in the cell-type x cell-type heatmap
- The macrophage-subset x niche heatmap shows co-occurrence of the three macrophage populations specific to the necrotic niche
- Control populations such as conventional DCs display niche-independent (uniform) hop distances, contrasting with the spatially specific macrophages
- The Delaunay neighbor graph is built successfully on the SpatialData object and cells_to_groups yields valid cell-type-to-niche links before mean_distance is computed

## Limitations

- Hop-distance statistics depend on upstream segmentation and annotation quality; errors in cell boundaries or niche assignment propagate into the colocalization results
- Demonstrated on the MERSCOPE liver dataset; transfer to other tissues/technologies (Xenium, CosMX, MACSima, PhenoCycler) requires valid pre-computed niches and cell-type labels
- Sopa's spatial analysis requires the data to be converted into a SpatialData object first; non-converted formats are not supported

## Evidence

- [discussion] As shown on the MERSCOPE liver dataset, we were able to annotate spatial-specific macrophages, particularly TREM2 macrophages, in the necrotic niche.: "As shown on the MERSCOPE liver dataset, we were able to annotate spatial-specific macrophages, particularly TREM2 macrophages, in the necrotic niche."
- [intro] Sopa converts any data into a SpatialData object, on which all of the six following tasks are performed.: "Sopa converts any data into a SpatialData object, on which all of the six following tasks are performed."
- [intro] we demonstrate Sopa's capabilities for geometric and spatial analysis on the MERSCOPE dataset by analyzing cell colocalization with regard to cell types and niches: [evidence span withheld — anchor status: not_found]
