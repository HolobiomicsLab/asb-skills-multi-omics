---
name: niche-polygon-vectorization
description: Use when you have a segmented SpatialData object with cells already assigned
  to spatial niches/cell-type groups and need polygons plus geometric statistics (area,
  perimeter, roundness, occurrence) per niche compartment.
when_to_use_negative:
- Cells have not yet been segmented and aggregated — run Sopa's segmentation/aggregation
  pipeline first, since spatial-omics libraries often rely on already-segmented data
- Niche/cell-type group labels are absent or unreliable — vectorization of arbitrary
  bins will not reproduce biologically meaningful compartment geometry
- The analysis goal is raw transcript-level segmentation (e.g., Baysor/Proseg run)
  rather than geometry of pre-defined niche compartments
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Sopa
  role: 'Provides the geometric-analysis API: sopa.spatial.spatial_neighbors, sopa.spatial.cells_to_groups,
    sopa.spatial.vectorize_niches, and sopa.spatial.niches_geometry_stats'
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Unified data framework holding the segmented MERSCOPE liver dataset on which
    niche grouping and vectorization are performed
  repo: https://github.com/scverse/spatialdata
- name: Shapely
  role: Polygon geometry engine used by Sopa to represent vectorized niche borders
    for area/perimeter/roundness computation
- name: STAGATE
  role: Supplies spatial-niche/cell-type group assignments that are converted into
    polygons
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:d372db5bbc970c31f55f3db20fe7ace164bda8ec56d7ab4e6b76c53b909ff453
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
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# niche-polygon-vectorization

## Summary

Vectorize cell-type or STAGATE-derived spatial niches into Shapely polygons with Sopa and compute per-niche geometric statistics (occurrence count, area, perimeter, roundness), demonstrated on the MERSCOPE liver dataset to reproduce Fig. 4b patterns (highly-occurring, small, round vascular niches vs. a single unround stroma polygon).

## When to use

Use when you have a segmented SpatialData object with cells already assigned to spatial niches/cell-type groups and need polygons plus geometric statistics (area, perimeter, roundness, occurrence) per niche compartment.

## When NOT to use

- Cells have not yet been segmented and aggregated — run Sopa's segmentation/aggregation pipeline first, since spatial-omics libraries often rely on already-segmented data
- Niche/cell-type group labels are absent or unreliable — vectorization of arbitrary bins will not reproduce biologically meaningful compartment geometry
- The analysis goal is raw transcript-level segmentation (e.g., Baysor/Proseg run) rather than geometry of pre-defined niche compartments

## Inputs

- Segmented spatial-omics dataset readable as a SpatialData object (e.g., MERSCOPE liver data via sopa.io.merscope)
- Per-cell niche and cell-type group assignments (e.g., from STAGATE)
- Per-cell spatial coordinates

## Outputs

- Shapely polygon objects, one per niche compartment (from sopa.spatial.vectorize_niches)
- Per-niche geometric statistics table (occurrence count, area, perimeter, roundness), e.g., niche_geometry_stats.csv
- Area/roundness distribution plots of niches

## How to apply

Load the technology-specific dataset (e.g., MERSCOPE liver via sopa.io.merscope) into a SpatialData object, then group cells into niches using sopa.spatial.spatial_neighbors and sopa.spatial.cells_to_groups with STAGATE-derived niche/cell-type assignments. Vectorize each niche into a Shapely polygon with sopa.spatial.vectorize_niches, which uses a Delaunay-graph border-simples approach to extract niche borders. Then run sopa.spatial.niches_geometry_stats to compute per-niche occurrence count, polygon area, perimeter, and roundness, and save the table (e.g., niche_geometry_stats.csv). The rationale, per the Sopa paper, is that geometric analysis of cell colocalization with respect to cell types and niches supports biological discoveries such as tumour-microenvironment morphology characterization.

## Related tools

- **Sopa** (Provides the geometric-analysis API: sopa.spatial.spatial_neighbors, sopa.spatial.cells_to_groups, sopa.spatial.vectorize_niches, and sopa.spatial.niches_geometry_stats) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Unified data framework holding the segmented MERSCOPE liver dataset on which niche grouping and vectorization are performed) — https://github.com/scverse/spatialdata
- **Shapely** (Polygon geometry engine used by Sopa to represent vectorized niche borders for area/perimeter/roundness computation)
- **STAGATE** (Supplies spatial-niche/cell-type group assignments that are converted into polygons)

## Examples

```
import sopa; sdata = sopa.io.merscope(path); polygons = sopa.spatial.vectorize_niches(sdata, bins_key='niche'); stats = sopa.spatial.niches_geometry_stats(sdata, bins_key='niche', polygons_key='niche')
```

## Evaluation signals

- niche_geometry_stats.csv exists and contains occurrence, area, perimeter, and roundness fields for each niche
- Vascular niches show the expected pattern: high occurrence, small area and perimeter, high roundness
- The stroma appears as a single occurrence with a highly unround polygon, matching the reported Fig. 4b pattern
- Each niche yields exactly one valid (non-empty) Shapely polygon after Delaunay border extraction
- Area/roundness distribution plot of niches can be generated without errors from the stats table

## Limitations

- Polygon quality depends on prior segmentation; Sopa notes existing libraries often rely on already-segmented data, so segmentation errors propagate into niche geometry
- The Delaunay-graph border-simples border extraction can produce unround polygons for large irregular compartments (e.g., stroma), which is expected but complicates roundness comparisons
- Sopa's integration of future methods is contingent on publication and validation, so alternative niche-detection or geometry tools may not yet be supported

## Evidence

- [abstract] Built upon the universal SpatialData framework, Sopa optimizes tasks like segmentation, transcript/channel aggregation, annotation, and geometric/spatial analysis.: "Built upon the universal SpatialData framework, Sopa optimizes tasks like segmentation, transcript/channel aggregation, annotation, and geometric/spatial analysis."
- [intro] we demonstrate Sopa's capabilities for geometric and spatial analysis on the MERSCOPE dataset by analyzing cell colocalization with regard to cell types and niches: [evidence span withheld — anchor status: not_found]
- [intro] Sopa converts any data into a SpatialData object, on which all of the six following tasks are performed.: "Sopa converts any data into a SpatialData object, on which all of the six following tasks are performed."
