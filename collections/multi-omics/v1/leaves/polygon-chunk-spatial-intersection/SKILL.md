---
name: polygon-chunk-spatial-intersection
description: Use when a segmentation produced cell polygons over a large on-disk multi-channel
  image and per-cell channel intensities must be averaged without loading the whole
  image into memory.
when_to_use_negative:
- Input is already a cell-by-channel intensity/feature table — averaging over polygons
  is redundant
- The image is small enough to fit in memory and raster masks are already available,
  so dense mask averaging is simpler and equally efficient
- Segmentation output is transcript-based only (e.g. Baysor) with no image channels
  to average
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Sopa
  role: Pipeline that implements fast channel averaging by combining geometry operations
    on polygon boundaries with image chunk lazy loading; provides sopa.io.toy_dataset
    and the sopa.aggregate min_intensity_ratio filter
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Universal framework holding the converted input data (lazy multi-channel image,
    cell boundary polygons, table) on which the chunked aggregation runs
  repo: https://github.com/scverse/spatialdata
- name: shapely
  role: Polygon geometry operations and spatial queries used to find cell polygons
    intersecting each image chunk's bounds
- name: dask
  role: Lazy loading backend of the image array so only per-cell bounding-box regions
    are loaded into memory
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:80d04696a6fa4129f5a89125210d01c65fe10403c13e8461f5cd44c28fe26825
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

# polygon-chunk-spatial-intersection

## Summary

Computes per-cell staining intensity (channel averaging) in a memory-efficient way by combining shapely-based spatial intersection of cell boundary polygons with image chunk lazy loading, instead of storing and averaging over full numpy raster masks.

## When to use

Use when a segmentation produced cell polygons over a large on-disk multi-channel image and per-cell channel intensities must be averaged without loading the whole image into memory.

## When NOT to use

- Input is already a cell-by-channel intensity/feature table — averaging over polygons is redundant
- The image is small enough to fit in memory and raster masks are already available, so dense mask averaging is simpler and equally efficient
- Segmentation output is transcript-based only (e.g. Baysor) with no image channels to average

## Inputs

- Segmented cell boundary polygons (e.g. in a SpatialData shapes element such as cell_boundaries)
- On-disk multi-channel image as a SpatialData object with a dask-backed lazy image array
- min_intensity_ratio threshold for the low-fluorescence-intensity cell filter

## Outputs

- channel_intensities.csv — per-cell averaged channel intensities table
- memory_validation.txt — comparison of chunked polygon-based vs dense numpy-mask peak memory usage
- Validation comparison showing chunked polygon-based mean intensities match dense numpy-mask values for a cell subset

## How to apply

Load the segmented cell boundary polygons and the on-disk multi-channel image as a SpatialData object (a toy dataset via sopa.io.toy_dataset is acceptable for validation). Iterate over image chunks; for each chunk, use shapely-based spatial queries to find the cell polygons intersecting the chunk bounds. For each intersecting cell, extract only the per-cell bounding-box region from the lazy dask-backed image array rather than loading the full image, rasterize the cell polygon locally within its bounding box, and average each channel's intensity over the rasterized pixels, applying the min_intensity_ratio filter to discard cells with low fluorescence intensity. Assemble the per-cell averaged channel intensities into a table (channel_intensities.csv); this mirrors Sopa's aggregation step in which 'we average the staining intensities and count the transcripts inside each cell, allowing further tasks such as annotation.' Validate by recomputing mean intensities for a subset of cells with a dense numpy-mask averaging approach and recording peak memory usage of both approaches.

## Related tools

- **Sopa** (Pipeline that implements fast channel averaging by combining geometry operations on polygon boundaries with image chunk lazy loading; provides sopa.io.toy_dataset and the sopa.aggregate min_intensity_ratio filter) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Universal framework holding the converted input data (lazy multi-channel image, cell boundary polygons, table) on which the chunked aggregation runs) — https://github.com/scverse/spatialdata
- **shapely** (Polygon geometry operations and spatial queries used to find cell polygons intersecting each image chunk's bounds)
- **dask** (Lazy loading backend of the image array so only per-cell bounding-box regions are loaded into memory)

## Examples

```
import sopa; sdata = sopa.io.toy_dataset(); sopa.aggregate(sdata, image_key='image', shapes_key='cell_boundaries', min_intensity_ratio=0.5)
```

## Evaluation signals

- Recomputed mean intensities for a subset of cells with a dense numpy-mask averaging approach match the chunked polygon-based values
- Peak memory usage recorded in memory_validation.txt is substantially lower for the chunked polygon-based approach than for the dense mask approach
- channel_intensities.csv contains one row per retained cell with one averaged value per image channel
- Cells with low fluorescence intensity are discarded when min_intensity_ratio is set, and no discarded cells appear in the output table
- SpatialData object round-trips: the image remains lazy (dask-backed) and cell polygons remain a shapes element after the run

## Limitations

- Conventional raster-mask storage of cell boundaries demands significant memory; the polygon+chunk approach exists to avoid this, so dense-mask comparisons are for validation only
- min_intensity_ratio filtering discards cells with low fluorescence intensity, which may remove weakly stained but real cells
- The workflow is demonstrated on a generated toy dataset (sopa.io.toy_dataset) whose runtime is 'a few minutes'; behavior on much larger images with millions of cells requires the memory-efficient design to be respected (no full-image loads)

## Evidence

- [intro] we average the staining intensities and count the transcripts inside each cell, allowing further tasks such as annotation: "we average the staining intensities and count the transcripts inside each cell, allowing further tasks such as annotation."
- [methods] A min_intensity_ratio value to remove cells with a low fluorescence intensity: "A [min_intensity_ratio](..//api/aggregation/#sopa.aggregate) value to remove cells with a low fluorescence intensity"
- [intro] the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts: "the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts."
- [intro] Sopa converts any data into a SpatialData object, on which all of the six following tasks are performed: "Sopa converts any data into a SpatialData object, on which all of the six following tasks are performed."
- [methods] For this tutorial, we use a generated dataset. You can expect a total runtime of a few minutes.: "For this tutorial, we use a generated dataset. You can expect a total runtime of a few minutes."
