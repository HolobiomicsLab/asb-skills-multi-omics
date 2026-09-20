---
name: xenium-explorer-data-conversion
description: Use when a Sopa pipeline run has produced a SpatialData zarr store (e.g.
  tuto.zarr) and the goal is to visualize cells, boundaries and image in the Xenium
  Explorer.
when_to_use_negative:
- The data is not image-based spatial omics readable by Sopa (e.g. non-spatial single-cell
  data), since the converter maps Sopa's SpatialData output into the Explorer format
- You only need programmatic downstream analysis and never interactive visualization
  — working directly with the SpatialData object or adata.h5ad is sufficient
- The pipeline has not yet completed segmentation and aggregation, so cell boundaries
  and gene counts do not exist to be converted
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Sopa
  role: Provides the explorer converter (sopa.io.explorer.write / write_image and
    the `sopa explorer write` CLI) that generates Xenium Explorer-compatible output
    files
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Universal framework holding the processed pipeline output that is read via
    spatialdata.read_zarr before conversion
  repo: https://github.com/scverse/spatialdata
- name: Xenium
  role: 10x Genomics platform whose Explorer software is the target visualizer consuming
    experiment.xenium and the converted files
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:9e6d3eadb5866a8b269bebf92d21ad032c226ac4efd9cf445800cc45aa582f63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - Sopa
  - SpatialData
  - Xenium
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# xenium-explorer-data-conversion

## Summary

Converts Sopa's processed SpatialData output into the six Xenium Explorer-compatible files (pyramidal OME-TIFF image, cell boundary polygons, cell categories, experiment.xenium, etc.) so the Explorer can serve as a unified visualizer for any image-based spatial omics technology.

## When to use

Use when a Sopa pipeline run has produced a SpatialData zarr store (e.g. tuto.zarr) and the goal is to visualize cells, boundaries and image in the Xenium Explorer.

## When NOT to use

- The data is not image-based spatial omics readable by Sopa (e.g. non-spatial single-cell data), since the converter maps Sopa's SpatialData output into the Explorer format
- You only need programmatic downstream analysis and never interactive visualization — working directly with the SpatialData object or adata.h5ad is sufficient
- The pipeline has not yet completed segmentation and aggregation, so cell boundaries and gene counts do not exist to be converted

## Inputs

- SpatialData zarr store produced by the Sopa pipeline (e.g. tuto.zarr) containing image, transcripts, cell boundaries and aggregated table
- Microscopy image (converted to pyramidal .ome.tif by write_image)
- AnnData gene-count table after aggregation/annotation (optional, for adata.h5ad output)

## Outputs

- tuto.explorer directory with experiment.xenium project file
- Pyramidal OME-TIFF image (single .ome.tif file when is_dir=False)
- Cell boundary polygon files
- Cell category (annotation) files
- tuto.explorer/adata.h5ad gene-count AnnData table

## How to apply

Load the processed SpatialData object with spatialdata.read_zarr from the zarr directory (e.g. tuto.zarr). First convert the microscopy image into the pyramidal .ome.tif format required by the Xenium Explorer using sopa.io.explorer.write_image (with is_dir=False for a single image file). Then run sopa.io.explorer.write (CLI: `sopa explorer write tuto.zarr`) to create the tuto.explorer directory containing experiment.xenium, cell boundary polygons, and cell category files. For time efficiency, split the conversion into a first pass saving only the image (`--mode "+i" --no-save-h5ad`) and a second pass saving boundaries and cell categories after aggregation/annotation (`--mode "-i"`). Finally save the gene-count table as tuto.explorer/adata.h5ad so downstream analyses can use the AnnData object instead of the full SpatialData object.

## Related tools

- **Sopa** (Provides the explorer converter (sopa.io.explorer.write / write_image and the `sopa explorer write` CLI) that generates Xenium Explorer-compatible output files) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Universal framework holding the processed pipeline output that is read via spatialdata.read_zarr before conversion) — https://github.com/scverse/spatialdata
- **Xenium** (10x Genomics platform whose Explorer software is the target visualizer consuming experiment.xenium and the converted files)

## Examples

```
sopa explorer write tuto.zarr
```

## Evaluation signals

- The tuto.explorer directory contains experiment.xenium and adata.h5ad after conversion
- The pyramidal image is written as a single .ome.tif file when is_dir=False (not a directory of tiles)
- The converted project loads successfully in the Xenium Explorer with image, cell boundaries and cell categories visible
- The two-pass split (`--mode "+i" --no-save-h5ad` then `--mode "-i"`) yields the same final file set as a single-pass conversion
- adata.h5ad gene-count table matches the aggregated AnnData in the SpatialData object

## Limitations

- The conversion is a post-hoc export: visualizer files must be regenerated (or split into image and non-image passes) if segmentation, aggregation or annotation outputs change
- Sopa's outputs target the Xenium Explorer format; compatibility relies on the converter maintaining the Explorer's expected file structure
- Converting very large images can be time-consuming, which is why the article recommends the two-pass image/boundaries split for efficiency

## Evidence

- [abstract] visualizer files output: "Its output includes user-friendly web reports and visualizer files, as well as comprehensive data files for in-depth analysis."
- [intro] SpatialData conversion: "Sopa converts any data into a SpatialData object, on which all of the six following tasks are performed."
- [abstract] unified visualizer claim: "We introduce Sopa ( https://github.com/gustaveroussy/sopa ), a technology-invariant, memory-efficient pipeline with a unified visualizer for all image-based spatial omics."
- [discussion] standardized outputs: "At the end of the pipeline, it produces standardized outputs, which ease exploration and visualization."
