---
name: memory-footprint-benchmarking
description: Use when you must quantify or validate the memory savings of Sopa's chunk-wise
  polygon-based channel averaging over full raster-mask loading, e.g. when adopting
  the aggregation step for large multi-channel images with millions of cells.
when_to_use_negative:
- Input data has already been aggregated into a cell-by-channel intensity table —
  no boundary geometry or image chunking is needed
- The image is small enough to fit fully in memory as a numpy array, where the dense-mask
  approach adds no memory burden
- 'Transcript-only platforms with no protein staining: the min_intensity_ratio intensity
  filter requires fluorescence image channels'
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Sopa
  role: Pipeline whose polygon-based, memory-efficient channel aggregation is benchmarked;
    provides aggregate() with the min_intensity_ratio filter
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Unified data framework holding the lazy dask-backed image and cell boundary
    polygons used for chunked queries
  repo: https://github.com/scverse/spatialdata
- name: shapely
  role: Geometry library used for spatial queries of cell polygons intersecting each
    image chunk's bounds and local rasterization
- name: dask
  role: Lazy-loading backend that defers image memory loading until per-cell bounding-box
    regions are needed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:c3534fe6036c09f6839326af70a2943f901015cc450f651aad799e9df94137b1
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# memory-footprint-benchmarking

## Summary

Benchmark the peak memory footprint of Sopa's chunked, polygon-based channel aggregation against a conventional dense numpy raster-mask averaging approach, demonstrating that deferring image loading via dask-backed chunk queries keeps per-cell staining-intensity computation memory-efficient on large spatial-omics images.

## When to use

Use when you must quantify or validate the memory savings of Sopa's chunk-wise polygon-based channel averaging over full raster-mask loading, e.g. when adopting the aggregation step for large multi-channel images with millions of cells.

## When NOT to use

- Input data has already been aggregated into a cell-by-channel intensity table — no boundary geometry or image chunking is needed
- The image is small enough to fit fully in memory as a numpy array, where the dense-mask approach adds no memory burden
- Transcript-only platforms with no protein staining: the min_intensity_ratio intensity filter requires fluorescence image channels

## Inputs

- SpatialData object containing segmented cell boundary polygons (GeoDataFrame) and a lazy dask-backed multi-channel image
- Toy dataset generated via sopa.io.toy_dataset for validation
- min_intensity_ratio threshold for the low-fluorescence-intensity cell filter

## Outputs

- channel_intensities.csv with per-cell averaged channel intensities
- memory_validation.txt recording peak memory usage of chunked polygon-based vs dense numpy-mask averaging and agreement of the mean intensities

## How to apply

Load segmented cell boundary polygons and the on-disk multi-channel image into a SpatialData object (a sopa.io.toy_dataset image is acceptable for validation). Iterate over image chunks, use shapely-based spatial queries to find cell polygons intersecting each chunk's bounds, and for each intersecting cell extract only the per-cell bounding-box region from the lazy (dask-backed) image array instead of loading the full image. Rasterize the cell polygon locally within its bounding box, average each channel's intensity over the rasterized pixels, and apply the min_intensity_ratio filter to discard cells with low fluorescence intensity. Assemble per-cell averaged channel intensities into a table written as channel_intensities.csv. Validate by recomputing mean intensities for a cell subset with a dense numpy-mask averaging approach and recording peak memory usage of both approaches in memory_validation.txt; the chunked polygon-based values should match the dense-mask values while using less peak memory.

## Related tools

- **Sopa** (Pipeline whose polygon-based, memory-efficient channel aggregation is benchmarked; provides aggregate() with the min_intensity_ratio filter) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Unified data framework holding the lazy dask-backed image and cell boundary polygons used for chunked queries) — https://github.com/scverse/spatialdata
- **shapely** (Geometry library used for spatial queries of cell polygons intersecting each image chunk's bounds and local rasterization)
- **dask** (Lazy-loading backend that defers image memory loading until per-cell bounding-box regions are needed)

## Examples

```
import sopa; sdata = sopa.io.toy_dataset(); sopa.aggregate(sdata, min_intensity_ratio=0.5)
```

## Evaluation signals

- Per-cell mean channel intensities from the chunked polygon-based approach match a dense numpy-mask recomputation on a subset of cells
- Peak memory usage recorded in memory_validation.txt is lower for the chunked polygon-based approach than for dense numpy-mask averaging
- channel_intensities.csv contains one row per cell passing the min_intensity_ratio filter, with one averaged value per image channel
- Cells with low fluorescence intensity are absent from the output table, confirming the min_intensity_ratio filter was applied

## Limitations

- Most existing spatial-omics methods are not designed for large images with millions of cells and demand high-memory computational clusters; this skill exists to quantify that contrast, not to remove per-cell geometry computation costs entirely
- Segmentation boundaries derived from polygon conversion can carry artefacts that the dense-mask recompute must mirror for a fair comparison
- Cells with very low fluorescence intensity are discarded by min_intensity_ratio, so intensity statistics only cover retained cells

## Evidence

- [results] Sopa implements fast channel averaging on cell boundaries by combining geometry operations on polygon boundaries with image chunk lazy loading: [evidence span withheld — anchor status: not_found]
- [intro] we average the staining intensities and count the transcripts inside each cell: "we average the staining intensities and count the transcripts inside each cell, allowing further tasks such as annotation."
- [methods] A [min_intensity_ratio](..//api/aggregation/#sopa.aggregate) value to remove cells with a low fluorescence intensity: "A [min_intensity_ratio](..//api/aggregation/#sopa.aggregate) value to remove cells with a low fluorescence intensity"
- [intro] Most existing methods are not designed to handle large images with millions of cells. Their usage typically demands high-performance computational clusters with substantial memory resources: [evidence span withheld — anchor status: not_found]
- [abstract] a technology-invariant, memory-efficient pipeline with a unified visualizer for all image-based spatial omics: "We introduce Sopa ( https://github.com/gustaveroussy/sopa ), a technology-invariant, memory-efficient pipeline with a unified visualizer for all image-based spatial omics."
