---
name: cell-type-hop-distance-computation
description: Use when you have an annotated SpatialData object (cell boundaries, cell-type
  labels, niche assignments) and need to test whether specific cell populations exclusively
  interact with themselves or are spatially restricted to particular niches.
when_to_use_negative:
- Input is raw image or transcript data that has not yet been segmented, annotated
  with cell types, or assigned to niches — Sopa requires the annotated SpatialData
  object as a prerequisite
- The analysis goal is transcript- or expression-level statistics rather than geometric/spatial
  colocalization between cell-type and niche labels
- You need physical distance (e.g., microns) rather than hop-distance on the Delaunay
  neighbor graph — this skill measures graph hops
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Sopa
  role: Provides spatial_neighbors (Delaunay graph), cells_to_groups, mean_distance,
    and niches_geometry_stats used to compute the hop-distance statistics
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Unified data framework holding the annotated cell boundaries, cell-type labels,
    and niche assignments that feed the spatial analysis
  repo: https://github.com/scverse/spatialdata
- name: MERSCOPE
  role: Source technology of the liver dataset on which the cell-type/niche colocalization
    analysis was demonstrated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:bbf24b50c2ea15725e61c1f7d5d9eceff3829bfd7e5cf12299f911aa71ecd96a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - MERSCOPE
  - Sopa
  - SpatialData
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cell-type-hop-distance-computation

## Summary

Computes cell-type × cell-type and cell-type × niche mean hop distances on a SpatialData cell-neighbor graph (Delaunay) to quantify colocalization, as Sopa does on the MERSCOPE liver dataset where LRP1, CEBP, and TREM2 macrophages co-localize specifically in the necrotic niche.

## When to use

Use when you have an annotated SpatialData object (cell boundaries, cell-type labels, niche assignments) and need to test whether specific cell populations exclusively interact with themselves or are spatially restricted to particular niches.

## When NOT to use

- Input is raw image or transcript data that has not yet been segmented, annotated with cell types, or assigned to niches — Sopa requires the annotated SpatialData object as a prerequisite
- The analysis goal is transcript- or expression-level statistics rather than geometric/spatial colocalization between cell-type and niche labels
- You need physical distance (e.g., microns) rather than hop-distance on the Delaunay neighbor graph — this skill measures graph hops

## Inputs

- Annotated SpatialData object with cell boundaries (polygons), cell-type labels, and niche assignments (e.g., MERSCOPE liver dataset)

## Outputs

- Cell-type × cell-type mean hop-distance matrix
- Cell-type (macrophage subset) × niche mean hop-distance matrix
- Niche-level geometry summaries (niches_geometry_stats)
- Heatmaps of mean hop distances (cell-type × cell-type and macrophage-subset × niche)

## How to apply

Load the annotated SpatialData object containing cell boundaries, cell-type labels, and niche assignments (e.g., the MERSCOPE liver dataset). Build the cell-neighbor graph with sopa.spatial.spatial_neighbors using Delaunay graph construction, then link cell types to their niches with sopa.spatial.cells_to_groups. Compute mean hop distances between cell-type and niche pairs with sopa.spatial.mean_distance, and derive niche-level summaries with niches_geometry_stats. Render heatmaps — one cell-type × cell-type, one macrophage-subset (LRP1, CEBP, TREM2) × niche — and inspect whether populations of interest show minimal hop distance to themselves and co-localize in a specific niche (e.g., the necrotic niche) versus niche-independent populations such as conventional DCs.

## Related tools

- **Sopa** (Provides spatial_neighbors (Delaunay graph), cells_to_groups, mean_distance, and niches_geometry_stats used to compute the hop-distance statistics) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Unified data framework holding the annotated cell boundaries, cell-type labels, and niche assignments that feed the spatial analysis) — https://github.com/scverse/spatialdata
- **MERSCOPE** (Source technology of the liver dataset on which the cell-type/niche colocalization analysis was demonstrated)

## Examples

```
import sopa.spatial; adata = sopa.spatial.spatial_neighbors(adata, delaunay=True); adata = sopa.spatial.cells_to_groups(adata, cell_type_key, niche_key); sopa.spatial.mean_distance(adata)
```

## Evaluation signals

- The mean hop-distance matrices are symmetric and finite for all cell-type pairs, with diagonal (self-interaction) values minimal for exclusive populations
- LRP1, CEBP, and TREM2 macrophages show minimal hop distance to themselves in the cell-type × cell-type heatmap and co-localize specifically in the necrotic niche in the subset × niche heatmap
- A control population (conventional DCs) displays niche-independent, roughly uniform hop distances across niches, confirming the analysis is not biased toward niche restriction
- Every cell is assigned to a cell type and a niche after cells_to_groups, so no cells are dropped between the neighbor graph and the mean_distance computation

## Limitations

- Hop distances depend on Delaunay graph construction of the cell-neighbor graph; results reflect graph topology rather than absolute physical distance
- The analysis requires prior cell-type annotation and niche assignment; these steps constrain the interpretation (Sopa's spatial-specific macrophage annotation, e.g., TREM2, was itself derived on this dataset)
- Inference is restricted to the niche definitions in the input object; colocalization conclusions are only as good as those niche assignments

## Evidence

- [intro] we demonstrate Sopa's capabilities for geometric and spatial analysis on the MERSCOPE dataset by analyzing cell colocalization with regard to cell types and niches: [evidence span withheld — anchor status: not_found]
- [discussion] As shown on the MERSCOPE liver dataset, we were able to annotate spatial-specific macrophages, particularly TREM2 macrophages, in the necrotic niche.: "As shown on the MERSCOPE liver dataset, we were able to annotate spatial-specific macrophages, particularly TREM2 macrophages, in the necrotic niche."
- [methods] Built on top of [SpatialData](https://github.com/scverse/spatialdata), Sopa enables processing: "Built on top of [SpatialData](https://github.com/scverse/spatialdata), Sopa enables processing"
- [readme] # SpatialData: an open and universal framework for processing spatial omics data.: [evidence span withheld — anchor status: not_found]
