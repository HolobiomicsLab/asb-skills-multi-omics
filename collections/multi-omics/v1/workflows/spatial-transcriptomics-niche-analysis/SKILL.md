---
name: spatial-transcriptomics-niche-analysis-workflow
description: 'Use when image-based spatial transcriptomics includes transcript coordinates,
  staining channels and an aligned histology image, and needs resolved cell boundaries,
  per-cell measurements and image-derived spatial niches.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 6
  member_skills:
  - spatial-omics-data-loading
  - spatial-omics-multilayer-image-alignment
  - spatial-transcriptomics-image-segmentation
  - transcript-based-cell-segmentation
  - overlapping-patch-tiling-strategy
  - spatial-omics-patch-based-image-segmentation
  - segmentation-conflict-resolution-across-patches
  - cell-boundary-intersection-over-min-area-quantification
  - spatial-transcriptomics-gene-count-aggregation
  - baysor-transcript-segmentation
  - channel-intensity-aggregation
  - polygon-chunk-spatial-intersection
  - per-cell-channel-intensity-quantification
  - cell-polygon-rasterization
  - spatial-niche-clustering-and-vectorization
  - spatial-niche-geometry-statistics
  - niche-cell-colocalization-analysis
  - cell-to-niche-group-assignment
  - xenium-explorer-data-conversion
  - cell-boundary-polygon-export
  - pyramidal-ome-tif-image-export
  member_tools:
  - MERSCOPE
  - Sopa
  - SpatialData
  - uv
  - Baysor
  - Cellpose
  - dask
  - shapely
  - Xenium
  coverage_gaps: []
  derived_from_workflows: []
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/multi-omics/v1
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# Image-Based Spatial Transcriptomics and Niche Analysis

## Summary

Load spatial images and transcripts, resolve segmentation across patches, aggregate cell measurements, derive image-based niches and export a spatial analysis store with geometry tables and visualization files.


## When to use

Use when image-based spatial transcriptomics includes transcript coordinates, staining channels and an aligned histology image, and needs resolved cell boundaries, per-cell measurements and image-derived spatial niches.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — load

**Goal:** spatial images and transcript coordinates to a unified persistent store

**EDAM operation:** http://edamontology.org/operation_2409

**Inputs:** transcript-table, image/staining, image/aligned-histology · **Outputs:** zarr/spatial-raw

**Candidate leaf skills:** `spatial-omics-data-loading` (primary), `spatial-omics-multilayer-image-alignment`, `spatial-transcriptomics-image-segmentation`

**Tools (primary):** MERSCOPE, Sopa, SpatialData, uv

**Other candidate tools:** Xenium, Hydra, Napari, Python, SPArrOW

**Grounding:** 2 KB(s); DOIs: 10.1038/s41467-024-48981-z, 10.1101/2024.07.04.601829

### Stage 2 — segment

**Goal:** transcript coordinates to resolved cell boundary polygons

**EDAM operation:** http://edamontology.org/operation_3799

**Inputs:** zarr/spatial-raw · **Outputs:** zarr/spatial-segmented

**Candidate leaf skills:** `transcript-based-cell-segmentation` (primary), `overlapping-patch-tiling-strategy`, `spatial-omics-patch-based-image-segmentation`, `segmentation-conflict-resolution-across-patches`, `cell-boundary-intersection-over-min-area-quantification`

**Tools (primary):** Baysor, Cellpose, MERSCOPE, Sopa, SpatialData

**Other candidate tools:** Xenium

**Grounding:** 1 KB(s); DOIs: 10.1038/s41467-024-48981-z

### Stage 3 — count_transcripts

**Goal:** cell polygons and transcripts to a filtered cell-by-gene matrix

**EDAM operation:** http://edamontology.org/operation_3799

**Inputs:** zarr/spatial-segmented · **Outputs:** zarr/spatial-gene-counts

**Candidate leaf skills:** `spatial-transcriptomics-gene-count-aggregation` (primary), `baysor-transcript-segmentation`

**Tools (primary):** MERSCOPE, Sopa, SpatialData

**Other candidate tools:** Baysor

**Grounding:** 1 KB(s); DOIs: 10.1038/s41467-024-48981-z

### Stage 4 — quantify_channels

**Goal:** retained cells and staining images to per-cell channel intensities

**EDAM operation:** http://edamontology.org/operation_3799

**Inputs:** zarr/spatial-gene-counts · **Outputs:** zarr/spatial-multimodal

**Candidate leaf skills:** `channel-intensity-aggregation` (primary), `polygon-chunk-spatial-intersection`, `per-cell-channel-intensity-quantification`, `cell-polygon-rasterization`

**Tools (primary):** Sopa, SpatialData, dask, shapely

**Other candidate tools:** MERSCOPE, Xenium, sopa.io.toy_dataset

**Grounding:** 1 KB(s); DOIs: 10.1038/s41467-024-48981-z

### Stage 5 — niches

**Goal:** aligned cell-centered image patches to niche labels and geometry tables

**EDAM operation:** http://edamontology.org/operation_3432

**Inputs:** zarr/spatial-multimodal · **Outputs:** zarr/spatial-niches, tsv/niche-geometry

**Candidate leaf skills:** `spatial-niche-clustering-and-vectorization` (primary), `spatial-niche-geometry-statistics`, `niche-cell-colocalization-analysis`, `cell-to-niche-group-assignment`

**Tools (primary):** MERSCOPE, Sopa, SpatialData, Xenium

**Other candidate tools:** STAGATE, Shapely

**Grounding:** 1 KB(s); DOIs: 10.1038/s41467-024-48981-z

### Stage 6 — export

**Goal:** analyzed spatial store to reusable cell data and visualization files

**EDAM operation:** http://edamontology.org/operation_0337

**Inputs:** zarr/spatial-niches, tsv/niche-geometry · **Outputs:** zarr/spatial-niches, h5ad, directory/explorer, tsv/niche-geometry

**Candidate leaf skills:** `xenium-explorer-data-conversion` (primary), `cell-boundary-polygon-export`, `pyramidal-ome-tif-image-export`

**Tools (primary):** Sopa, SpatialData, Xenium


**Grounding:** 1 KB(s); DOIs: 10.1038/s41467-024-48981-z

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
