---
name: macrophage-subset-spatial-interaction-analysis
description: Use when a task requires quantifying cell-type/cell-type and cell-type/niche
  spatial interaction via hop distances on a SpatialData object, e.g. verifying niche-specific
  macrophage co-localization in MERSCOPE liver data.
when_to_use_negative:
- Input is raw image/transcript data that has not yet been segmented and annotated
  — Sopa's spatial analysis operates on an already-annotated SpatialData object
- The analysis goal is segmentation or transcript/channel aggregation rather than
  post-annotation geometric/spatial statistics
- Data is not image-based spatial omics (e.g. bulk or single-cell dissociated data
  without spatial coordinates)
tools:
- name: Sopa
  role: 'Provides the spatial statistics API: sopa.spatial.spatial_neighbors (Delaunay
    graph), sopa.spatial.cells_to_groups, sopa.spatial.mean_distance, niches_geometry_stats'
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Universal framework backing the annotated MERSCOPE liver SpatialData object
    consumed by Sopa's spatial analysis
  repo: https://github.com/scverse/spatialdata
- name: MERSCOPE
  role: Source spatial-transcriptomics technology for the liver dataset on which the
    hop-distance analysis is demonstrated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:05bc22e060a1a3eeae4278fcf98b44e595a1753f2196f83c44e60536aa509c9b
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
yes_rate: 0.8571428571428571
n_claims: 7
---

# macrophage-subset-spatial-interaction-analysis

## Summary

Reproduce Sopa's cell-type and niche hop-distance spatial statistics on the MERSCOPE liver dataset to test whether macrophage subpopulations (LRP1, CEBP, TREM2) preferentially interact with themselves and co-localize in the necrotic niche, using a Delaunay cell-neighbor graph.

## When to use

Use when a task requires quantifying cell-type/cell-type and cell-type/niche spatial interaction via hop distances on a SpatialData object, e.g. verifying niche-specific macrophage co-localization in MERSCOPE liver data.

## When NOT to use

- Input is raw image/transcript data that has not yet been segmented and annotated — Sopa's spatial analysis operates on an already-annotated SpatialData object
- The analysis goal is segmentation or transcript/channel aggregation rather than post-annotation geometric/spatial statistics
- Data is not image-based spatial omics (e.g. bulk or single-cell dissociated data without spatial coordinates)

## Inputs

- Annotated SpatialData object of the MERSCOPE liver dataset with cell boundaries, cell-type labels, and niche assignments
- Cell-type annotation table for macrophage subpopulations (LRP1, CEBP, TREM2) and conventional DCs
- Niche assignment column (e.g. necrotic niche labels)

## Outputs

- Cell-neighbor graph (Delaunay) over cells
- Mean hop-distance matrices between cell-type pairs and between macrophage subsets and niches
- Niche-level geometry summaries from niches_geometry_stats
- Cell-type x cell-type and macrophage-subset x niche heatmaps

## How to apply

Load the annotated SpatialData object (MERSCOPE liver dataset) containing cell boundary polygons, cell-type labels, and niche assignments. Build the cell-neighbor graph with sopa.spatial.spatial_neighbors using Delaunay graph construction, then link each cell type to its niche with sopa.spatial.cells_to_groups. Compute mean hop distances between cell-type and niche pairs with sopa.spatial.mean_distance, and obtain niche-level geometric summaries with niches_geometry_stats. Render two heatmaps — one cell-type x cell-type and one macrophage-subset x niche — and check that LRP1/CEBP/TREM2 macrophages show minimal hop distance to themselves and co-localize specifically in the necrotic niche, while conventional DCs show niche-independent distances. The rationale is that Sopa's geometric/spatial analysis on SpatialData-backed inputs enables colocalization statistics at millions-of-cells scale without dedicated proprietary tools.

## Related tools

- **Sopa** (Provides the spatial statistics API: sopa.spatial.spatial_neighbors (Delaunay graph), sopa.spatial.cells_to_groups, sopa.spatial.mean_distance, niches_geometry_stats) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Universal framework backing the annotated MERSCOPE liver SpatialData object consumed by Sopa's spatial analysis) — https://github.com/scverse/spatialdata
- **MERSCOPE** (Source spatial-transcriptomics technology for the liver dataset on which the hop-distance analysis is demonstrated)

## Examples

```
import sopa.spatial; adj = sopa.spatial.spatial_neighbors(sdata, method='delaunay'); groups = sopa.spatial.cells_to_groups(sdata); d = sopa.spatial.mean_distance(sdata, adj, groups)
```

## Evaluation signals

- LRP1, CEBP, and TREM2 macrophages show minimal mean hop distance to themselves (self-interaction dominance) in the cell-type x cell-type heatmap
- The macrophage-subset x niche heatmap shows the three macrophage populations co-occur specifically in the necrotic niche
- Conventional DCs display niche-independent (approximately uniform) hop distances across niches, serving as a negative-control pattern
- Delaunay neighbor graph is built without errors on the full MERSCOPE liver SpatialData object and mean_distance returns a complete cell-type/niche distance matrix

## Limitations

- Bayes-like niche-specificity conclusions depend on prior annotation quality: niche and cell-type labels must already be present in the SpatialData object
- Hop-distance statistics rely on Delaunay graph construction, which can misrepresent neighborhoods in sparse or boundary regions
- Sopa's demonstrated macrophage result (TREM2 macrophages in the necrotic niche) was shown on the MERSCOPE liver dataset; transferability to other tissues/technologies is not established by this analysis

## Evidence

- [results] Macrophage co-localization finding: [evidence span withheld — anchor status: not_found]
- [discussion] MERSCOPE liver demonstration: "As shown on the MERSCOPE liver dataset, we were able to annotate spatial-specific macrophages, particularly TREM2 macrophages, in the necrotic niche."
- [intro] Spatial analysis goal: [evidence span withheld — anchor status: not_found]
- [abstract] SpatialData framework: "Built upon the universal SpatialData framework, Sopa optimizes tasks like segmentation, transcript/channel aggregation, annotation, and geometric/spatial analysis."
