---
name: spatial-heatmap-visualization
description: Use when a spatial-omics analysis has produced a cell-type or niche pairwise
  distance/interaction matrix (e.g. Sopa mean hop distances) and the pattern must
  be inspected or communicated as a heatmap.
when_to_use_negative:
- Input is raw imaging or transcript data not yet segmented, annotated with cell types,
  or assigned to niches — the heatmap step requires a labeled SpatialData table
- The analysis goal is a segmentation or annotation task (Cellpose/Baysor segmentation,
  transcript aggregation) rather than post-annotation spatial statistics
edam_operation: http://edamontology.org/operation_0337
tools:
- name: Sopa
  role: Provides sopa.spatial.spatial_neighbors, cells_to_groups, mean_distance, and
    niches_geometry_stats used to compute the hop-distance matrices visualized as
    heatmaps
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Unified data framework holding the annotated MERSCOPE SpatialData object (cell
    boundaries, labels, niches) that feeds the spatial statistics
  repo: https://github.com/scverse/spatialdata
- name: MERSCOPE
  role: Image-based spatial transcriptomics platform whose liver dataset supplies
    the cell-type/niche data analyzed in the heatmaps
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:4f1f5a8d67a4933429fb05add920af55f004e9f61666b9518a946797be860101
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - MERSCOPE
  - Sopa
  - SpatialData
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spatial-heatmap-visualization

## Summary

Renders heatmaps of mean hop distances computed from Sopa's cell-neighbor spatial statistics (e.g. cell-type x cell-type and macrophage-subset x niche matrices on the MERSCOPE liver dataset) to visually verify cell-type colocalization patterns such as LRP1/CEBP/TREM2 macrophage co-occurrence in the necrotic niche.

## When to use

Use when a spatial-omics analysis has produced a cell-type or niche pairwise distance/interaction matrix (e.g. Sopa mean hop distances) and the pattern must be inspected or communicated as a heatmap.

## When NOT to use

- Input is raw imaging or transcript data not yet segmented, annotated with cell types, or assigned to niches — the heatmap step requires a labeled SpatialData table
- The analysis goal is a segmentation or annotation task (Cellpose/Baysor segmentation, transcript aggregation) rather than post-annotation spatial statistics

## Inputs

- Annotated SpatialData object (e.g. MERSCOPE liver dataset) with cell boundaries, cell-type labels, and niche assignments
- Cell-neighbor graph (Delaunay) from sopa.spatial.spatial_neighbors
- Cell-type-to-niche group assignments from sopa.spatial.cells_to_groups
- Mean hop-distance matrix from sopa.spatial.mean_distance (cell-type x cell-type and cell-type x niche)

## Outputs

- Cell-type x cell-type mean hop-distance heatmap
- Macrophage-subset x niche mean hop-distance heatmap
- Niche-level geometry summary statistics (niches_geometry_stats)

## How to apply

After building the Delaunay cell-neighbor graph with sopa.spatial.spatial_neighbors and linking cell types to niches via sopa.spatial.cells_to_groups, compute the pairwise mean hop distances with sopa.spatial.mean_distance (optionally summarized per niche with niches_geometry_stats). Render the resulting matrices as two heatmaps: one cell-type x cell-type and one macrophage-subset x niche. Read the heatmap for minimal (self-)hop distances: on the MERSCOPE liver dataset, LRP1, CEBP, and TREM2 macrophages should show minimal hop distance to themselves and co-occurrence specific to the necrotic niche, while conventional DCs display niche-independent distances. Correctness is judged by whether the rendered heatmap reproduces these block-diagonal self-interaction and niche-specific co-localization signatures.

## Related tools

- **Sopa** (Provides sopa.spatial.spatial_neighbors, cells_to_groups, mean_distance, and niches_geometry_stats used to compute the hop-distance matrices visualized as heatmaps) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Unified data framework holding the annotated MERSCOPE SpatialData object (cell boundaries, labels, niches) that feeds the spatial statistics) — https://github.com/scverse/spatialdata
- **MERSCOPE** (Image-based spatial transcriptomics platform whose liver dataset supplies the cell-type/niche data analyzed in the heatmaps)

## Examples

```
import sopa.spatial; sopa.spatial.spatial_neighbors(sdata); sopa.spatial.cells_to_groups(sdata); dist = sopa.spatial.mean_distance(sdata); plot heatmap of dist
```

## Evaluation signals

- LRP1, CEBP, and TREM2 macrophage rows show minimal hop distance on the diagonal (self-interaction) in the cell-type x cell-type heatmap
- In the macrophage-subset x niche heatmap, the three macrophage populations co-occur specifically in the necrotic niche column
- Conventional DC rows display niche-independent (uniform) distances, serving as a negative control for the macrophage pattern
- Distance matrices are symmetric (for the cell-type x cell-type heatmap) and derived from the same Delaunay neighbor graph used by mean_distance

## Limitations

- Interpretation depends on upstream annotation and niche-clustering quality; mislabeled cell types or niches distort the hop-distance heatmaps
- The demonstrated biological pattern (necrotic-niche TREM2 macrophages) was shown on the MERSCOPE liver dataset; transfer to other tissues/technologies (Xenium, CosMX, MACSima, PhenoCycler) is not established by this analysis

## Evidence

- [abstract] Built upon the universal SpatialData framework, Sopa optimizes tasks like segmentation, transcript/channel aggregation, annotation, and geometric/spatial analysis.: "Built upon the universal SpatialData framework, Sopa optimizes tasks like segmentation, transcript/channel aggregation, annotation, and geometric/spatial analysis."
- [intro] we demonstrate Sopa's capabilities for geometric and spatial analysis on the MERSCOPE dataset by analyzing cell colocalization with regard to cell types and niches: [evidence span withheld — anchor status: not_found]
- [discussion] As shown on the MERSCOPE liver dataset, we were able to annotate spatial-specific macrophages, particularly TREM2 macrophages, in the necrotic niche.: "As shown on the MERSCOPE liver dataset, we were able to annotate spatial-specific macrophages, particularly TREM2 macrophages, in the necrotic niche."
