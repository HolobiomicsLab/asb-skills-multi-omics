---
name: spatial-omics-colocalization-analysis
description: Use when a segmented SpatialData object has cell-type or niche assignments
  (e.g. STAGATE clusters on MERSCOPE transcripts) and you need geometric/spatial statistics
  per niche rather than per cell.
when_to_use_negative:
- Input is raw transcripts or images without prior segmentation — Sopa requires already-segmented
  cells before geometric analysis
- The goal is per-cell segmentation, aggregation, or annotation rather than niche-level
  geometric/spatial analysis
- Only single-cell (non-spatial) expression tables are available; no cell coordinates
  exist to build the spatial neighbor graph
edam_operation: http://edamontology.org/operation_0390
tools:
- name: Sopa
  role: 'Provides the geometric-analysis API: sopa.spatial.spatial_neighbors, cells_to_groups,
    vectorize_niches, and niches_geometry_stats for colocalization of cell types and
    niches'
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Unified data representation that Sopa converts inputs into and on which all
    analysis tasks, including geometric/spatial analysis, are performed
  repo: https://github.com/scverse/spatialdata
- name: Shapely
  role: Polygon representation of vectorized niche compartments used for area/perimeter/roundness
    computation
- name: STAGATE
  role: Supplies the niche/cell-type group assignments that are grouped and vectorized
    into niches
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:42dfd1422b0632f67b02ba7c0f2b33642f93d326fa5559f2066a8f68141e5dd4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - STAGATE
  - Shapely
  - Sopa
  - SpatialData
  edam_operation: http://edamontology.org/operation_0390
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# spatial-omics-colocalization-analysis

## Summary

Vectorize STAGATE-derived spatial niches into Shapely polygons with Sopa and compute per-niche geometric statistics (occurrence, area, perimeter, roundness), enabling cell-type/niche colocalization analysis on image-based spatial omics data such as the MERSCOPE liver dataset.

## When to use

Use when a segmented SpatialData object has cell-type or niche assignments (e.g. STAGATE clusters on MERSCOPE transcripts) and you need geometric/spatial statistics per niche rather than per cell.

## When NOT to use

- Input is raw transcripts or images without prior segmentation — Sopa requires already-segmented cells before geometric analysis
- The goal is per-cell segmentation, aggregation, or annotation rather than niche-level geometric/spatial analysis
- Only single-cell (non-spatial) expression tables are available; no cell coordinates exist to build the spatial neighbor graph

## Inputs

- Segmented spatial-omics dataset as a SpatialData object (e.g. MERSCOPE liver data read via sopa.io.merscope)
- Per-cell niche/cell-type-group assignments (e.g. STAGATE-derived labels)
- Cell coordinate tables within the SpatialData object for neighbor-graph construction

## Outputs

- Shapely polygons, one per niche compartment (from sopa.spatial.vectorize_niches)
- Per-niche geometric statistics table (occurrence, area, perimeter, roundness), e.g. niche_geometry_stats.csv
- Area/roundness distribution plots of niches

## How to apply

Load the segmented dataset into a SpatialData object (e.g. sopa.io.merscope for MERSCOPE liver data), then build a cell-cell graph with sopa.spatial.spatial_neighbors and group cells into niches with sopa.spatial.cells_to_groups. Vectorize each niche into a Shapely polygon via sopa.spatial.vectorize_niches, using the Delaunay-graph border-simples approach to extract niche borders. Run sopa.spatial.niches_geometry_stats to compute occurrence count, polygon area, perimeter, and roundness per niche compartment, and save the table as niche_geometry_stats.csv. Interpretation: vascular niches show high occurrence, small area/perimeter, and high roundness, while the stroma appears as a single, highly unround polygon, matching the Fig. 4b pattern of the Sopa paper.

## Related tools

- **Sopa** (Provides the geometric-analysis API: sopa.spatial.spatial_neighbors, cells_to_groups, vectorize_niches, and niches_geometry_stats for colocalization of cell types and niches) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Unified data representation that Sopa converts inputs into and on which all analysis tasks, including geometric/spatial analysis, are performed) — https://github.com/scverse/spatialdata
- **Shapely** (Polygon representation of vectorized niche compartments used for area/perimeter/roundness computation)
- **STAGATE** (Supplies the niche/cell-type group assignments that are grouped and vectorized into niches)

## Examples

```
import sopa; sdata = sopa.io.merscope(path); sopa.spatial.niches_geometry_stats(sopa.spatial.vectorize_niches(sdata))
```

## Evaluation signals

- niche_geometry_stats.csv exists and contains occurrence, area, and roundness fields for each niche
- Vascular niches show high occurrence, small area and perimeter, and high roundness
- The stroma compartment appears as a single occurrence and is highly unround, reproducing the reported Fig. 4b pattern
- Each niche resolves to exactly one or more valid Shapely polygons after vectorization (no empty/degenerate geometries)
- Area/roundness distribution plots of niches can be produced from the stats table

## Limitations

- Current open-source analysis libraries, including this workflow, depend on already-segmented data; raw images/transcripts must pass through Sopa's segmentation and aggregation steps first
- Extra information layers beyond proteins or transcripts are not supported in the current technologies covered
- No software version numbers for Sopa or dependencies are reported in this section, so exact reproducibility of statistics may depend on the environment

## Evidence

- [intro] Geometric/spatial analysis capability: [evidence span withheld — anchor status: not_found]
- [intro] SpatialData as unified representation: "Sopa converts any data into a SpatialData object, on which all of the six following tasks are performed."
- [methods] Segmentation prerequisite: "Using a [min_area](../api/segmentation/#sopa.segmentation.cellpose) threshold to remove small cells (provide this argument to the segmentation methods)."
- [discussion] MERSCOPE liver biological validation: "As shown on the MERSCOPE liver dataset, we were able to annotate spatial-specific macrophages, particularly TREM2 macrophages, in the necrotic niche."
- [methods] Vectorization API support: "::: sopa.shapes.vectorize"
