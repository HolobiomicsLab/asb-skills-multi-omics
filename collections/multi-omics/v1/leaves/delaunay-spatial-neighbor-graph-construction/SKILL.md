---
name: delaunay-spatial-neighbor-graph-construction
description: Use when you have a segmented SpatialData object with cell boundaries,
  cell-type labels, and niche assignments, and you need cell-cell adjacency to quantify
  colocalization via hop distances.
when_to_use_negative:
- Input is not yet segmented or lacks cell-type/niche labels — Sopa's segmentation,
  aggregation, and annotation steps must run first (Cellpose/Baysor, transcript/channel
  aggregation)
- You need expression-based neighborhood domains rather than geometric adjacency —
  use Novae (same authors) instead of a pure Delaunay hop-distance analysis
- You need interaction distances in physical units with a fixed radius rather than
  graph hop counts — Delaunay hop distance is a topological, not micrometer-scale,
  proximity measure
tools:
- name: Sopa
  role: Provides sopa.spatial.spatial_neighbors (Delaunay graph construction), sopa.spatial.cells_to_groups,
    sopa.spatial.mean_distance, and niches_geometry_stats used for the hop-distance
    analysis
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Unified data framework holding the cell boundaries, cell tables, and niche
    assignments on which the neighbor graph is built
  repo: https://github.com/scverse/spatialdata
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:78308a77080af6031805a2e20b18901fa79e211226b8a905944eac5a4ded0b34
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
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# delaunay-spatial-neighbor-graph-construction

## Summary

Builds a cell-neighbor graph over segmented spatial-omics cells (e.g., the MERSCOPE liver dataset in Sopa) using Delaunay triangulation via sopa.spatial.spatial_neighbors, so that hop-distance statistics between cell types and niches can be computed on a technology-invariant SpatialData representation.

## When to use

Use when you have a segmented SpatialData object with cell boundaries, cell-type labels, and niche assignments, and you need cell-cell adjacency to quantify colocalization via hop distances.

## When NOT to use

- Input is not yet segmented or lacks cell-type/niche labels — Sopa's segmentation, aggregation, and annotation steps must run first (Cellpose/Baysor, transcript/channel aggregation)
- You need expression-based neighborhood domains rather than geometric adjacency — use Novae (same authors) instead of a pure Delaunay hop-distance analysis
- You need interaction distances in physical units with a fixed radius rather than graph hop counts — Delaunay hop distance is a topological, not micrometer-scale, proximity measure

## Inputs

- Annotated SpatialData object (e.g., MERSCOPE liver) with cell boundary polygons
- Cell-by-gene count table with cell-type annotation labels (AnnData table in sdata.tables)
- Niche assignment / niche label table per cell

## Outputs

- Delaunay cell-neighbor graph over segmented cells
- Mean hop-distance matrix between cell-type pairs
- Cell-type x niche mean hop-distance heatmap (macrophage subsets vs. necrotic niche)
- Niche-level geometry summaries from niches_geometry_stats

## How to apply

First convert/load the annotated dataset (e.g., MERSCOPE liver) as a SpatialData object containing cell boundary polygons and a cell-by-gene table with cell-type and niche labels. Build the cell-neighbor graph with sopa.spatial.spatial_neighbors, which constructs a Delaunay graph from the cell centroids/polygons; Delaunay triangulation is chosen because it gives an adjacency structure without a user-tuned radius threshold, which suits Sopa's memory-efficient, technology-invariant design. Link each cell to its niche with sopa.spatial.cells_to_groups, then compute mean hop distances between cell-type and niche pairs with sopa.spatial.mean_distance and niche-level summaries via niches_geometry_stats. The rationale is that hop distance on the Delaunay graph captures proximity: LRP1, CEBP, and TREM2 macrophages should show minimal hop distance to themselves and co-occurrence specific to the necrotic niche, while conventional DCs should display niche-independent distances.

## Related tools

- **Sopa** (Provides sopa.spatial.spatial_neighbors (Delaunay graph construction), sopa.spatial.cells_to_groups, sopa.spatial.mean_distance, and niches_geometry_stats used for the hop-distance analysis) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Unified data framework holding the cell boundaries, cell tables, and niche assignments on which the neighbor graph is built) — https://github.com/scverse/spatialdata

## Examples

```
import sopa; sopa.spatial.spatial_neighbors(sdata); sopa.spatial.cells_to_groups(sdata, cell_type_key='cell_type', niche_key='niche')
```

## Evaluation signals

- The Delaunay graph covers all segmented cells with no isolated components unexpectedly large relative to tissue density
- LRP1, CEBP, and TREM2 macrophage subpopulations show minimal hop distance to themselves (near-diagonal minimum in the cell-type x cell-type heatmap)
- The macrophage-subset x niche heatmap shows LRP1/CEBP/TREM2 co-occurrence specific to the necrotic niche
- Conventional DCs display niche-independent (roughly uniform) hop distances, serving as a negative-control cell type
- Hop-distance results are reproducible across the technology-invariant SpatialData object regardless of input platform

## Limitations

- Conclusions are specific to the annotated MERSCOPE liver dataset; necrotic-niche macrophage colocalization depends on the upstream annotation quality
- Hop distances depend on the Delaunay triangulation and the correctness of cell boundaries from segmentation; artifacts in merged masks propagate into the graph
- Sopa's future segmentation/annotation methods can only be added once published and validated, constraining methodological updates

## Evidence

- [other] Delaunay graph construction step for cell-neighbor graph: [evidence span withheld — anchor status: not_found]
- [abstract] Sopa's spatial/geometric analysis capability on SpatialData framework: "Built upon the universal SpatialData framework, Sopa optimizes tasks like segmentation, transcript/channel aggregation, annotation, and geometric/spatial analysis."
- [intro] Demonstration of cell colocalization analysis on MERSCOPE: [evidence span withheld — anchor status: not_found]
- [discussion] Biological validation of the niche-level analysis: "As shown on the MERSCOPE liver dataset, we were able to annotate spatial-specific macrophages, particularly TREM2 macrophages, in the necrotic niche."
