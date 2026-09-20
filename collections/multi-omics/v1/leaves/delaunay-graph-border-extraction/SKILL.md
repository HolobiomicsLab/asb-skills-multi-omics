---
name: delaunay-graph-border-extraction
description: Use when cell-level spatial data has been grouped into niches (e.g.,
  via STAGATE) and you need contiguous polygon shapes per niche to compute geometric
  properties like area, perimeter, and roundness.
when_to_use_negative:
- Cells have not yet been assigned to niches or cell-type groups — the Delaunay border
  extraction operates on grouped cells, not raw segmentation masks
- Input is already polygonized (e.g., cell_boundaries shapes from Proseg/SpatialData
  output) and only overlap resolution or expansion (sopa.shapes.remove_overlap, sopa.shapes.expand_radius)
  is needed
- The analysis goal is transcript-level segmentation (Cellpose/Baysor/Proseg) rather
  than post-annotation niche geometry
tools:
- name: Sopa
  role: Provides sopa.spatial.spatial_neighbors, sopa.spatial.cells_to_groups, sopa.spatial.vectorize_niches
    (Delaunay-graph border extraction), and sopa.spatial.niches_geometry_stats
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Unified data framework holding the segmented MERSCOPE dataset on which niche
    vectorization and geometric analysis are performed
  repo: https://github.com/scverse/spatialdata
- name: Shapely
  role: Polygon geometry representation into which each niche is vectorized via sopa.spatial.vectorize_niches
- name: STAGATE
  role: Supplies the spatial-domain (niche) assignments that define the cell groups
    to be vectorized
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:5e5184b4e6e4c60451696604a5c4eeed592a664048ba32ef431e2c44223dfb48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - STAGATE
  - Shapely
  - Sopa
  - SpatialData
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Delaunay-graph border extraction for niche vectorization

## Summary

Extracts the borders of spatial-omics niches (e.g., STAGATE-identified compartments in MERSCOPE liver data) by vectorizing cell groups into Shapely polygons via a Delaunay-graph border-simples approach in Sopa, enabling per-niche geometric statistics such as occurrence, area, perimeter, and roundness.

## When to use

Use when cell-level spatial data has been grouped into niches (e.g., via STAGATE) and you need contiguous polygon shapes per niche to compute geometric properties like area, perimeter, and roundness.

## When NOT to use

- Cells have not yet been assigned to niches or cell-type groups — the Delaunay border extraction operates on grouped cells, not raw segmentation masks
- Input is already polygonized (e.g., cell_boundaries shapes from Proseg/SpatialData output) and only overlap resolution or expansion (sopa.shapes.remove_overlap, sopa.shapes.expand_radius) is needed
- The analysis goal is transcript-level segmentation (Cellpose/Baysor/Proseg) rather than post-annotation niche geometry

## Inputs

- Segmented spatial-omics dataset as a SpatialData object (e.g., MERSCOPE liver data read with sopa.io.merscope)
- Cell-level niche/cell-type group assignments (e.g., STAGATE-derived niche labels)
- Cell centroid coordinates used to build the spatial neighbor graph

## Outputs

- Shapely polygons, one per niche compartment, from sopa.spatial.vectorize_niches
- Per-niche geometry statistics table (occurrence, area, perimeter, roundness) saved as niche_geometry_stats.csv
- Area/roundness distribution plots of the vectorized niches

## How to apply

Load the segmented dataset (e.g., MERSCOPE liver data via sopa.io.merscope) as a SpatialData object, then build cell-type/niche groupings using sopa.spatial.spatial_neighbors together with sopa.spatial.cells_to_groups on the STAGATE-derived niche assignments. Apply sopa.spatial.vectorize_niches, which uses a Delaunay graph over cell positions and extracts border simplices to trace each niche's outer contour into a Shapely polygon. With the polygons in place, call sopa.spatial.niches_geometry_stats to compute per-niche occurrence count, polygon area, and roundness (a polygon is round if its area is close to that of a circle with equal perimeter). The rationale is that vectorizing niches turns discrete cell labels into continuous shapes so vascular-type compartments appear as many small, highly round polygons while broad compartments like stroma appear as single, highly unround polygons. Save the statistics table as niche_geometry_stats.csv and plot the area/roundness distributions to validate against the reported Fig. 4b pattern.

## Related tools

- **Sopa** (Provides sopa.spatial.spatial_neighbors, sopa.spatial.cells_to_groups, sopa.spatial.vectorize_niches (Delaunay-graph border extraction), and sopa.spatial.niches_geometry_stats) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Unified data framework holding the segmented MERSCOPE dataset on which niche vectorization and geometric analysis are performed) — https://github.com/scverse/spatialdata
- **Shapely** (Polygon geometry representation into which each niche is vectorized via sopa.spatial.vectorize_niches)
- **STAGATE** (Supplies the spatial-domain (niche) assignments that define the cell groups to be vectorized)

## Examples

```
import sopa.spatial as sp; niches = sp.vectorize_niches(sdata, key='niche'); stats = sp.niches_geometry_stats(niches); stats.to_csv('niche_geometry_stats.csv')
```

## Evaluation signals

- niche_geometry_stats.csv exists and contains occurrence, area, and roundness fields per niche
- Vascular niches show the expected pattern: high occurrence, small area and perimeter, and high roundness
- The stroma compartment resolves to a single occurrence and is highly unround
- The computed statistics reproduce the reported Fig. 4b pattern on the MERSCOPE liver dataset
- Area/roundness distribution plots of vectorized niches are generated without errors from sopa.spatial.vectorize_niches

## Limitations

- Border quality depends on the STAGATE niche assignments and the Delaunay-graph construction; misgrouped cells will distort polygon contours
- Sopa's support for new segmentation/annotation methods depends on their publication and validation, so niche-definition methods are not interchangeable without revalidation
- The geometric statistics reported are occurrence, area, perimeter, and roundness; other shape descriptors are not provided by this pipeline step

## Evidence

- [other] Vectorize each niche into a Shapely polygon via sopa.spatial.vectorize_niches, using the Delaunay-graph border-simples approach to extract niche borders.: [evidence span withheld — anchor status: not_found]
- [methods] sopa.shapes.vectorize method evidence: "::: sopa.shapes.vectorize"
- [abstract] abstract framework quote: "Built upon the universal SpatialData framework, Sopa optimizes tasks like segmentation, transcript/channel aggregation, annotation, and geometric/spatial analysis."
- [intro] introduction research question on geometric analysis: [evidence span withheld — anchor status: not_found]
