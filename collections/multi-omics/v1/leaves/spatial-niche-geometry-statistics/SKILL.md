---
name: spatial-niche-geometry-statistics
description: Use when a segmented image-based spatial-omics dataset (e.g. MERSCOPE
  read via sopa.io.merscope) already has niche/cell-type group assignments and you
  need per-niche polygon geometry (area, perimeter, roundness, occurrence) statistics.
when_to_use_negative:
- Input is raw imaging or transcript data that has not yet been segmented and annotated
  (niche grouping requires per-cell assignments downstream of segmentation/aggregation).
- Cells have no niche/STAGATE group labels yet — build niches first via sopa.spatial.spatial_neighbors
  and sopa.spatial.cells_to_groups.
- The goal is cell-level rather than niche-level geometry (e.g. removing small cells
  via min_area or expanding cell masks with sopa.shapes.expand_radius).
tools:
- name: Sopa
  role: Provides sopa.spatial.spatial_neighbors, cells_to_groups, vectorize_niches,
    and niches_geometry_stats used to group cells into niches, vectorize niche borders,
    and compute geometric statistics
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Unified data framework holding the segmented MERSCOPE dataset that Sopa's
    geometric/spatial analysis operates on
  repo: https://github.com/scverse/spatialdata
- name: Shapely
  role: Polygon representation used by sopa.spatial.vectorize_niches to build niche
    polygons from Delaunay-graph border-simples
- name: STAGATE
  role: Supplies the niche/cell-type group assignments that define which cells are
    grouped into each spatial niche before vectorization
- name: MERSCOPE
  role: Image-based spatial transcriptomics technology whose liver dataset is the
    demonstration input for the niche geometry statistics
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:53c9d1262bd672fc191905a9e8acd0cef6665f5f19f311cc65ab92cabe28f110
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - MERSCOPE
  - STAGATE
  - Shapely
  - Sopa
  - SpatialData
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spatial-niche-geometry-statistics

## Summary

Converts STAGATE-defined spatial niches in a segmented SpatialData object into Shapely polygons and computes per-niche geometric statistics (occurrence count, area, perimeter, roundness), as demonstrated by Sopa on the MERSCOPE liver dataset to reproduce the Fig. 4b niche-morphology pattern.

## When to use

Use when a segmented image-based spatial-omics dataset (e.g. MERSCOPE read via sopa.io.merscope) already has niche/cell-type group assignments and you need per-niche polygon geometry (area, perimeter, roundness, occurrence) statistics.

## When NOT to use

- Input is raw imaging or transcript data that has not yet been segmented and annotated (niche grouping requires per-cell assignments downstream of segmentation/aggregation).
- Cells have no niche/STAGATE group labels yet — build niches first via sopa.spatial.spatial_neighbors and sopa.spatial.cells_to_groups.
- The goal is cell-level rather than niche-level geometry (e.g. removing small cells via min_area or expanding cell masks with sopa.shapes.expand_radius).

## Inputs

- Segmented SpatialData object (e.g. MERSCOPE liver dataset read via sopa.io.merscope) with per-cell annotations
- STAGATE-derived niche/cell-type group assignments per cell
- Cell centroids/spatial coordinates for building spatial neighbors (Delaunay graph)

## Outputs

- Per-niche Shapely polygons from sopa.spatial.vectorize_niches
- niche_geometry_stats.csv with occurrence, area, perimeter, and roundness per niche
- Area/roundness distribution plot of niches

## How to apply

Load the segmented dataset into a SpatialData object (e.g. MERSCOPE liver data via sopa.io.merscope) and group cells into niches using STAGATE-derived niche/cell-type assignments together with sopa.spatial.spatial_neighbors and sopa.spatial.cells_to_groups. Vectorize each niche into a Shapely polygon with sopa.spatial.vectorize_niches, using the Delaunay-graph border-simples approach to extract niche borders. Then call sopa.spatial.niches_geometry_stats to compute occurrence count, polygon area, perimeter, and roundness per niche, saving the result as niche_geometry_stats.csv and plotting the area/roundness distribution. The rationale is that geometric properties distinguish niche morphology biologically: on the MERSCOPE liver data, vascular niches show high occurrence, small area/perimeter and high roundness, while the stroma appears as a single large, highly unround polygon.

## Related tools

- **Sopa** (Provides sopa.spatial.spatial_neighbors, cells_to_groups, vectorize_niches, and niches_geometry_stats used to group cells into niches, vectorize niche borders, and compute geometric statistics) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Unified data framework holding the segmented MERSCOPE dataset that Sopa's geometric/spatial analysis operates on) — https://github.com/scverse/spatialdata
- **Shapely** (Polygon representation used by sopa.spatial.vectorize_niches to build niche polygons from Delaunay-graph border-simples)
- **STAGATE** (Supplies the niche/cell-type group assignments that define which cells are grouped into each spatial niche before vectorization)
- **MERSCOPE** (Image-based spatial transcriptomics technology whose liver dataset is the demonstration input for the niche geometry statistics)

## Examples

```
import sopa.spatial as sp; polys = sp.vectorize_niches(sdata); stats = sp.niches_geometry_stats(polys); stats.to_csv('niche_geometry_stats.csv')
```

## Evaluation signals

- niche_geometry_stats.csv exists and contains occurrence, area, and roundness fields for every niche
- Vascular niches show high occurrence, small area and perimeter, and high roundness
- The stroma appears as a single occurrence with a large, highly unround polygon, matching the reported Fig. 4b pattern
- Each niche vectorizes to exactly one or more valid Shapely polygons extracted via the Delaunay border-simples approach
- Area/roundness distribution plot renders without missing-niche entries

## Limitations

- The geometric analysis demonstrated is tied to niche groupings from upstream methods (STAGATE); Sopa supports adding future segmentation/annotation methods only once published and validated.
- Delaunay-graph border extraction can be sensitive to sparse or noisy cell placements at niche borders, which can distort polygon area/perimeter/roundness.
- No software version numbers for Sopa or its dependencies are reported in the article section.

## Evidence

- [abstract] Built upon the universal SpatialData framework, Sopa optimizes tasks like segmentation, transcript/channel aggregation, annotation, and geometric/spatial analysis.: "Built upon the universal SpatialData framework, Sopa optimizes tasks like segmentation, transcript/channel aggregation, annotation, and geometric/spatial analysis."
- [intro] geometric and spatial analysis on MERSCOPE: [evidence span withheld — anchor status: not_found]
- [methods] min_area cell filtering: "Using a [min_area](../api/segmentation/#sopa.segmentation.cellpose) threshold to remove small cells"
- [readme] SpatialData object shapes for cell polygons: [evidence span withheld — anchor status: not_found]
