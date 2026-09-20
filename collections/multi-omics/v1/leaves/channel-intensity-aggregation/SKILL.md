---
name: channel-intensity-aggregation
description: Use when you have segmented cell polygons plus an on-disk multi-channel
  image in a SpatialData object and need per-cell mean channel intensities without
  loading full raster masks into memory.
when_to_use_negative:
- Input is already a cell x channel feature/annotation table — no aggregation is needed
- The image is small enough that full raster-mask averaging is trivially cheap, and
  you do not need Sopa's memory-efficient chunked path
- The task is transcript (not staining) aggregation — use Sopa's transcript counting
  / min_transcripts path instead
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: Sopa
  role: Provides the aggregation API that averages staining intensities per cell with
    min_intensity_ratio filtering, in a memory-efficient pipeline
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Unified data framework holding the cell boundary polygons (shapes) and dask-backed
    image chunks
  repo: https://github.com/scverse/spatialdata
- name: shapely
  role: Spatial queries to find cell polygons intersecting each image chunk and local
    polygon rasterization within bounding boxes
- name: dask
  role: Lazy loading of image chunks so only per-cell bounding-box regions are materialized
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:4f975ffd3c0f0f65c4cd171298f973b64fa258681abf80d1d412d474bd380089
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - Sopa
  - SpatialData
  - dask
  - shapely
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# channel-intensity-aggregation

## Summary

Per-cell staining-intensity aggregation as implemented in Sopa: instead of storing full raster cell masks, cell boundaries are kept as Shapely polygons and per-channel intensities are averaged by lazily loading only each cell's bounding-box region from a dask-backed image chunk, keeping memory use low for images with millions of cells.

## When to use

Use when you have segmented cell polygons plus an on-disk multi-channel image in a SpatialData object and need per-cell mean channel intensities without loading full raster masks into memory.

## When NOT to use

- Input is already a cell x channel feature/annotation table — no aggregation is needed
- The image is small enough that full raster-mask averaging is trivially cheap, and you do not need Sopa's memory-efficient chunked path
- The task is transcript (not staining) aggregation — use Sopa's transcript counting / min_transcripts path instead

## Inputs

- Segmented cell boundary polygons (Shapely geometries, e.g. SpatialData shapes from converted segmentation masks)
- On-disk multi-channel fluorescence image as a dask-backed array inside a SpatialData object
- min_intensity_ratio threshold parameter for low-intensity cell filtering

## Outputs

- Per-cell averaged channel intensity table (cell x channel matrix, e.g. channel_intensities.csv)
- Filtered cell set with low-intensity cells removed via min_intensity_ratio

## How to apply

Load the segmented cell boundary polygons and the multi-channel image as a SpatialData object (e.g. via sopa.io.toy_dataset for validation). Iterate over image chunks; for each chunk use shapely-based spatial queries to find cell polygons intersecting the chunk bounds, then extract only each cell's bounding-box region from the lazy dask-backed array rather than the full image. Rasterize the cell polygon locally within its bounding box and average each channel over the rasterized pixels, applying the min_intensity_ratio filter to discard cells with low fluorescence intensity. Assemble per-cell averaged channel intensities into a table; this implements Sopa's aggregation step of 'averaging the staining intensities... inside each cell', enabling downstream annotation.

## Related tools

- **Sopa** (Provides the aggregation API that averages staining intensities per cell with min_intensity_ratio filtering, in a memory-efficient pipeline) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Unified data framework holding the cell boundary polygons (shapes) and dask-backed image chunks) — https://github.com/scverse/spatialdata
- **shapely** (Spatial queries to find cell polygons intersecting each image chunk and local polygon rasterization within bounding boxes)
- **dask** (Lazy loading of image chunks so only per-cell bounding-box regions are materialized)

## Examples

```
import sopa; sdata = sopa.io.toy_dataset(); sopa.aggregate(sdata, min_intensity_ratio=0.5)
```

## Evaluation signals

- Recompute mean intensities for a subset of cells with a dense numpy-mask averaging approach and verify the chunked polygon-based values match
- Record peak memory usage of both the chunked polygon approach and dense-mask averaging in memory_validation.txt; the chunked approach should use substantially less peak memory
- Output table has one row per retained cell and one column per image channel, with cells below the min_intensity_ratio threshold absent
- Every cell in the output corresponds to a polygon that intersects the processed image chunks (no missing/extra cell IDs)

## Limitations

- min_intensity_ratio discards cells with low fluorescence intensity, so weakly stained cells are excluded from the intensity table
- Averaging operates on locally rasterized bounding-box pixels, so results should be validated against dense-mask computation; polygon rasterization introduces discretization at cell edges
- Memory savings depend on deferring loading via dask-backed chunks; eager in-memory arrays forfeit the benefit

## Evidence

- [intro] we average the staining intensities and count the transcripts inside each cell, allowing further tasks such as annotation.: "we average the staining intensities and count the transcripts inside each cell, allowing further tasks such as annotation."
- [methods] A [min_intensity_ratio](..//api/aggregation/#sopa.aggregate) value to remove cells with a low fluorescence intensity: "A [min_intensity_ratio](..//api/aggregation/#sopa.aggregate) value to remove cells with a low fluorescence intensity"
- [abstract] We introduce Sopa ( https://github.com/gustaveroussy/sopa ), a technology-invariant, memory-efficient pipeline with a unified visualizer for all image-based spatial omics.: "We introduce Sopa ( https://github.com/gustaveroussy/sopa ), a technology-invariant, memory-efficient pipeline with a unified visualizer for all image-based spatial omics."
