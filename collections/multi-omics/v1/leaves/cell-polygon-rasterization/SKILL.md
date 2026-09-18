---
name: cell-polygon-rasterization
description: Use when per-cell staining-intensity (channel averaging) values must
  be computed for millions of cells over large multi-channel images where conventional
  raster-mask storage would exceed available memory.
when_to_use_negative:
- When cell masks are small enough that full-resolution numpy raster masks fit in
  memory and a simple mask-based mean is sufficient — the polygon chunking machinery
  adds complexity without memory benefit
- When the segmentation tool already emits a cell-by-channel intensity table (no aggregation
  needed)
- 'When cells have low fluorescence intensity overall: min_intensity_ratio filtering
  may remove them rather than quantify them'
tools:
- name: Sopa
  role: Provides the memory-efficient aggregation step (sopa.aggregate with min_intensity_ratio)
    that converts segmentation masks into polygons and averages staining intensities
    inside each cell
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Unified data framework holding the cell polygons and the dask-backed multi-channel
    image that enable chunk-wise lazy loading
  repo: https://github.com/scverse/spatialdata
- name: sopa.io.toy_dataset
  role: Generated dataset used to validate the channel-averaging workflow without
    real instrument data
  repo: https://github.com/gustaveroussy/sopa
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:3832a62cafb50992119ccae2102c1914c31736e3a8818c2e4fed043787b08c69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - Sopa
  - SpatialData
  - dask
  - sopa.io.toy_dataset
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cell-polygon-rasterization

## Summary

A memory-efficient channel-intensity aggregation strategy from Sopa in which segmentation masks are stored as cell boundary polygons and staining intensities are averaged by rasterizing each polygon locally within its bounding box over a lazy (dask-backed) image, instead of holding full numpy raster masks in memory.

## When to use

Use when per-cell staining-intensity (channel averaging) values must be computed for millions of cells over large multi-channel images where conventional raster-mask storage would exceed available memory.

## When NOT to use

- When cell masks are small enough that full-resolution numpy raster masks fit in memory and a simple mask-based mean is sufficient — the polygon chunking machinery adds complexity without memory benefit
- When the segmentation tool already emits a cell-by-channel intensity table (no aggregation needed)
- When cells have low fluorescence intensity overall: min_intensity_ratio filtering may remove them rather than quantify them

## Inputs

- SpatialData object containing cell boundary polygons (from Cellpose/Baysor segmentation masks) and a multi-channel image stored as a lazy dask-backed array (e.g. via sopa.io.toy_dataset or an on-disk Xenium/MERSCOPE/CosMX image)

## Outputs

- cell-by-channel table of averaged staining intensities (channel_intensities.csv) after min_intensity_ratio filtering
- memory_validation.txt reporting peak memory usage of the chunked polygon approach versus dense numpy-mask averaging

## How to apply

First convert the segmented boundaries and multi-channel on-disk image into a SpatialData object, since Sopa converts any data into a SpatialData object on which all pipeline tasks are performed. During aggregation, the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts. Rather than loading the whole image, iterate over image chunks, use shapely spatial queries to find cell polygons intersecting each chunk, and extract only each cell's bounding-box region from the dask-backed image array. Rasterize the polygon locally within that bounding box and average each channel's intensity over the rasterized pixels, then count transcripts similarly. Apply the min_intensity_ratio threshold (exposed by sopa.aggregate) to discard cells with low fluorescence intensity, assembling the surviving per-cell values into a cell-by-channel table. Validate by recomputing mean intensities for a cell subset with dense numpy-mask averaging and confirming the chunked polygon-based values match while recording peak memory of both approaches.

## Related tools

- **Sopa** (Provides the memory-efficient aggregation step (sopa.aggregate with min_intensity_ratio) that converts segmentation masks into polygons and averages staining intensities inside each cell) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Unified data framework holding the cell polygons and the dask-backed multi-channel image that enable chunk-wise lazy loading) — https://github.com/scverse/spatialdata
- **sopa.io.toy_dataset** (Generated dataset used to validate the channel-averaging workflow without real instrument data) — https://github.com/gustaveroussy/sopa

## Examples

```
import sopa; sdata = sopa.io.toy_dataset(); sopa.aggregate(sdata, min_intensity_ratio=0.2)
```

## Evaluation signals

- Chunked polygon-based mean intensities match dense numpy-mask mean intensities recomputed for a subset of cells (within numerical tolerance)
- Cells below the min_intensity_ratio fluorescence threshold are excluded from the output table
- Cell polygons are merged over overlapping patches such that segmentation artefacts across patch boundaries are removed
- Peak memory usage recorded in memory_validation.txt is substantially lower for the chunked polygon approach than for dense raster-mask averaging
- Output is a valid cell-by-channel intensity table aligned with the cell ids of the SpatialData shapes

## Limitations

- Aggregation requires segmentation masks/polygons to exist first; the workflow depends on per-patch segmentation (Cellpose for images, Baysor for transcripts) completed beforehand
- min_intensity_ratio filtering removes cells with low fluorescence intensity, so dim cells are dropped rather than reported with noisy values

## Evidence

- [intro] polygons merged before aggregation: "the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts."
- [intro] aggregation definition: "we average the staining intensities and count the transcripts inside each cell, allowing further tasks such as annotation."
- [methods] min_intensity_ratio filter: "A [min_intensity_ratio](..//api/aggregation/#sopa.aggregate) value to remove cells with a low fluorescence intensity"
- [abstract] SpatialData basis: "Built upon the universal SpatialData framework, Sopa optimizes tasks like segmentation, transcript/channel aggregation, annotation, and geometric/spatial analysis."
