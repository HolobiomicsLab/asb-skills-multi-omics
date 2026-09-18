---
name: cell-to-niche-group-assignment
description: Use when a segmented SpatialData object (e.g. MERSCOPE liver cells with
  cell-type or niche labels) must be grouped into spatially contiguous niches and
  characterized geometrically.
when_to_use_negative:
- Input is raw image or transcript data that has not yet been segmented and aggregated
  into a per-cell table in a SpatialData object
- The goal is cell-level (single-cell) segmentation or annotation rather than grouping
  cells into higher-order spatial niches
- Only whole-tissue segmentation is needed — use sopa.segmentation.tissue instead
  of niche-level vectorization
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Sopa
  role: Provides spatial_neighbors, cells_to_groups, vectorize_niches, and niches_geometry_stats
    for niche grouping and geometric analysis
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Unified data framework holding the segmented cells, transcripts, and resulting
    niche polygons
  repo: https://github.com/scverse/spatialdata
- name: STAGATE
  role: Supplies the niche/cell-type group assignments that cells are mapped onto
    via the spatial-neighbor graph
- name: Shapely
  role: Polygon geometry backend used by vectorize_niches and by the area/perimeter/roundness
    computations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:b6ae1f184b177de4fde9d03522c77d5b3149163289e30e2f39e470d404443079
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - STAGATE
  - Shapely
  - Sopa
  - SpatialData
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cell-to-niche-group-assignment

## Summary

Assigns segmented cells to spatial niches by combining STAGATE-derived niche/cell-type labels with a spatial-neighbor graph in Sopa, then converts each niche into Shapely polygons via Delaunay-graph border simplices and computes per-niche geometric statistics (occurrence, area, perimeter, roundness).

## When to use

Use when a segmented SpatialData object (e.g. MERSCOPE liver cells with cell-type or niche labels) must be grouped into spatially contiguous niches and characterized geometrically.

## When NOT to use

- Input is raw image or transcript data that has not yet been segmented and aggregated into a per-cell table in a SpatialData object
- The goal is cell-level (single-cell) segmentation or annotation rather than grouping cells into higher-order spatial niches
- Only whole-tissue segmentation is needed — use sopa.segmentation.tissue instead of niche-level vectorization

## Inputs

- Segmented spatial-omics dataset as a SpatialData object (e.g. MERSCOPE liver read via sopa.io.merscope)
- Per-cell niche/cell-type group assignments from STAGATE
- Transcript/cell coordinate tables embedded in the SpatialData object

## Outputs

- Vectorized Shapely polygons, one per niche compartment (from sopa.spatial.vectorize_niches)
- Per-niche geometric statistics table (occurrence count, area, perimeter, roundness) saved as niche_geometry_stats.csv
- Area/roundness distribution plots of the niches

## How to apply

Load the segmented dataset into a SpatialData object (e.g. MERSCOPE liver read via sopa.io.merscope). Build the cell-neighbor graph with sopa.spatial.spatial_neighbors, then assign cells to niches using STAGATE-derived group assignments via sopa.spatial.cells_to_groups. Vectorize each niche compartment with sopa.spatial.vectorize_niches, which extracts niche borders from Delaunay-graph border simplices into Shapely polygons. Finally run sopa.spatial.niches_geometry_stats to obtain per-niche occurrence count, polygon area, perimeter, and roundness, and save the table as niche_geometry_stats.csv. The rationale is that geometric properties of niches (e.g. many small, round vascular compartments vs. a single large unround stroma) recapitulate spatial biology patterns such as those shown in the article's Fig. 4b.

## Related tools

- **Sopa** (Provides spatial_neighbors, cells_to_groups, vectorize_niches, and niches_geometry_stats for niche grouping and geometric analysis) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Unified data framework holding the segmented cells, transcripts, and resulting niche polygons) — https://github.com/scverse/spatialdata
- **STAGATE** (Supplies the niche/cell-type group assignments that cells are mapped onto via the spatial-neighbor graph)
- **Shapely** (Polygon geometry backend used by vectorize_niches and by the area/perimeter/roundness computations)

## Examples

```
import sopa; sdata = sopa.io.merscope(path); sopa.spatial.spatial_neighbors(sdata); sopa.spatial.cells_to_groups(sdata, groups_key="niche"); niches = sopa.spatial.vectorize_niches(sdata); stats = sopa.spatial.niches_geometry_stats(niches)
```

## Evaluation signals

- niche_geometry_stats.csv exists and contains occurrence, area, and roundness fields for every niche compartment
- Vascular niches show high occurrence, small area and perimeter, and high roundness
- The stroma compartment appears as a single occurrence and is highly unround
- The overall pattern reproduces the article's Fig. 4b area/roundness distribution on the MERSCOPE liver dataset
- Every niche label from the STAGATE grouping is represented by at least one vectorized polygon (no empty polygons)

## Limitations

- Niche border extraction depends on the Delaunay-graph border-simples approach, so niche polygon fidelity is limited by the density and correctness of the underlying cell-neighbor graph
- Results are only as good as the upstream STAGATE niche/cell-type assignments; mislabeled cells distort niche geometry statistics
- Sopa's supported segmentation/annotation methods expand only once new methods are published and validated, and downstream geometric analysis relies on already-segmented data

## Evidence

- [intro] mer scope geometric analysis: [evidence span withheld — anchor status: not_found]
- [methods] built on spatialdata: "Built on top of [SpatialData](https://github.com/scverse/spatialdata), Sopa enables processing"
- [discussion] merscope liver niches: "As shown on the MERSCOPE liver dataset, we were able to annotate spatial-specific macrophages, particularly TREM2 macrophages, in the necrotic niche."
- [abstract] sopa task coverage: "Built upon the universal SpatialData framework, Sopa optimizes tasks like segmentation, transcript/channel aggregation, annotation, and geometric/spatial analysis."
