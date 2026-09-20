---
name: lazy-image-bounding-box-extraction
description: Use when computing per-cell staining-intensity aggregates on large multi-channel
  images with polygon cell boundaries, where loading full raster masks would exhaust
  memory.
when_to_use_negative:
- Input is already an aggregated cell-by-channel feature table — no image or polygon
  geometry is present, so bounding-box extraction is unnecessary.
- The image is small enough that dense numpy raster-mask averaging fits comfortably
  in memory and a simpler dense-mask implementation suffices.
- No polygon representation of cell boundaries exists (e.g. only raw segmentation
  masks not yet vectorized) — Sopa converts masks to polygons before aggregation.
tools:
- name: Sopa
  role: Pipeline performing segmentation, conversion of masks to polygons, and channel
    aggregation with min_intensity_ratio filtering
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Universal data framework holding the on-disk lazy image and polygon cell boundaries
  repo: https://github.com/scverse/spatialdata
- name: shapely
  role: Polygon geometry and spatial queries to find cells intersecting each image
    chunk
- name: dask
  role: Lazy chunked loading of the image so only per-cell bounding boxes are materialized
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:4a6875b68c7d4068184efd94e510ac75fc7ff99222973d600fd4c1e9caf00cc4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - Sopa
  - SpatialData
  - dask
  - shapely
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# lazy-image-bounding-box-extraction

## Summary

Memory-efficient per-cell channel averaging in Sopa: instead of materializing full raster masks, cell-boundary polygons are spatially queried per image chunk and only each cell's small bounding-box region is loaded from the dask-backed (lazy) image, where the polygon is rasterized locally and channel intensities are averaged.

## When to use

Use when computing per-cell staining-intensity aggregates on large multi-channel images with polygon cell boundaries, where loading full raster masks would exhaust memory.

## When NOT to use

- Input is already an aggregated cell-by-channel feature table — no image or polygon geometry is present, so bounding-box extraction is unnecessary.
- The image is small enough that dense numpy raster-mask averaging fits comfortably in memory and a simpler dense-mask implementation suffices.
- No polygon representation of cell boundaries exists (e.g. only raw segmentation masks not yet vectorized) — Sopa converts masks to polygons before aggregation.

## Inputs

- SpatialData object containing cell-boundary polygons (shapes) from segmentation
- On-disk multi-channel image stored as a dask-backed (lazy) array in the SpatialData object
- min_intensity_ratio threshold parameter (e.g. passed to sopa.aggregate)

## Outputs

- Per-cell averaged channel intensities (cell x channel table, e.g. channel_intensities.csv)
- Cell table with low-intensity cells removed by the min_intensity_ratio filter
- memory_validation.txt recording peak memory of chunked vs dense-mask approaches (for validation)

## How to apply

Load the segmented cell-boundary polygons and the on-disk multi-channel image together in a SpatialData object (a toy dataset via sopa.io.toy_dataset suffices for validation). Iterate over image chunks and, for each chunk, use shapely spatial queries to find cell polygons intersecting the chunk bounds. For each intersecting cell, slice only its bounding-box region from the lazy dask-backed array — deferring memory loading until needed — then rasterize the polygon within that bounding box and average each channel's intensity over the rasterized pixels. Apply the min_intensity_ratio filter (sopa.aggregate) to discard cells with low fluorescence intensity, and assemble per-cell channel means into a table. Rationale: Sopa optimizes transcript/channel aggregation on the universal SpatialData framework, so polygon-plus-chunk geometry avoids the memory footprint of full numpy raster masks.

## Related tools

- **Sopa** (Pipeline performing segmentation, conversion of masks to polygons, and channel aggregation with min_intensity_ratio filtering) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Universal data framework holding the on-disk lazy image and polygon cell boundaries) — https://github.com/scverse/spatialdata
- **shapely** (Polygon geometry and spatial queries to find cells intersecting each image chunk)
- **dask** (Lazy chunked loading of the image so only per-cell bounding boxes are materialized)

## Examples

```
import sopa; sdata = sopa.io.toy_dataset(); sopa.aggregate(sdata, min_intensity_ratio=0.0)
```

## Evaluation signals

- Recompute mean intensities for a subset of cells with a dense numpy-mask averaging approach; chunked polygon-based values must match the dense-mask values.
- Peak memory usage of the chunked polygon approach (recorded in memory_validation.txt) should be substantially lower than the dense-mask approach.
- channel_intensities.csv should contain one row per retained cell and one column per image channel, with cells below min_intensity_ratio removed.
- Cells whose polygons fall outside all image chunks should not appear in the output table.
- Values should be finite (no NaNs) for all retained cells whose bounding boxes lie within the image extent.

## Limitations

- Sopa's integration of new segmentation/annotation methods is contingent on their publication and validation, so upstream polygon sources must already be supported (Cellpose, Baysor, Proseg).
- Cells discarded by min_intensity_ratio have low fluorescence intensity and are excluded from downstream annotation, which may bias toward brighter stains.
- Accuracy depends on polygon fidelity: segmentation masks are converted into polygons and merged over patches to remove artefacts, so patch-boundary artefacts can propagate into per-cell averages.

## Evidence

- [abstract] Sopa is a memory-efficient, technology-invariant pipeline: "We introduce Sopa ( https://github.com/gustaveroussy/sopa ), a technology-invariant, memory-efficient pipeline with a unified visualizer for all image-based spatial omics."
- [intro] Aggregation averages staining intensities per cell: "we average the staining intensities and count the transcripts inside each cell, allowing further tasks such as annotation."
- [intro] SpatialData object is the unified input: "Sopa converts any data into a SpatialData object, on which all of the six following tasks are performed."
- [methods] min_intensity_ratio filters low-intensity cells: "A [min_intensity_ratio](..//api/aggregation/#sopa.aggregate) value to remove cells with a low fluorescence intensity"
- [intro] Masks converted to polygons before aggregation: "the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts."
