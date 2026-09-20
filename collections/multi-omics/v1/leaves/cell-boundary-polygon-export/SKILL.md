---
name: cell-boundary-polygon-export
description: Use when a Sopa-processed SpatialData object (e.g. a tuto.zarr store)
  must be converted into Xenium Explorer input files — experiment.xenium, pyramidal
  .ome.tif image, cell boundary polygons, and cell category files — for visualization.
when_to_use_negative:
- Data has not been processed by Sopa (no aggregated cell-by-gene table or merged
  boundary polygons yet) — run segmentation and aggregation first.
- The goal is visualization in a viewer other than the Xenium Explorer, or the input
  is already a Xenium-native experiment (no conversion needed).
- Only raw image/transcript files are available and the full Sopa pipeline (patching,
  Cellpose/Baysor segmentation, transcript counting) still needs to run — this skill
  is the final export step, not the pipeline itself.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: Sopa
  role: Provides the converter (sopa.io.explorer.write / write_image and the `sopa
    explorer write` CLI) that transforms the Sopa SpatialData output into Xenium Explorer-compatible
    files.
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Framework on which the Sopa output is built; the zarr store (read with spatialdata.read_zarr)
    is the converter's input object.
  repo: https://github.com/scverse/spatialdata
- name: Xenium
  role: 'Target visualizer: the exported experiment.xenium project loads directly
    in the Xenium Explorer to display cells, transcripts, and image across technologies
    such as Xenium, MERSCOPE, CosMX, MACSima, and PhenoCycler.'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:b01fceacac940a5fbe01f77acfe353a811434a2fcfb8d9d5e35b7a8faf9b3c87
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
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cell-boundary-polygon-export

## Summary

Export Sopa's processed SpatialData object (cell boundary polygons, image, and cell category/gene-count files) into the six Xenium Explorer-compatible output files, so that any image-based spatial omics technology readable by Sopa can be visualized in one shared, optimized visualizer.

## When to use

Use when a Sopa-processed SpatialData object (e.g. a tuto.zarr store) must be converted into Xenium Explorer input files — experiment.xenium, pyramidal .ome.tif image, cell boundary polygons, and cell category files — for visualization.

## When NOT to use

- Data has not been processed by Sopa (no aggregated cell-by-gene table or merged boundary polygons yet) — run segmentation and aggregation first.
- The goal is visualization in a viewer other than the Xenium Explorer, or the input is already a Xenium-native experiment (no conversion needed).
- Only raw image/transcript files are available and the full Sopa pipeline (patching, Cellpose/Baysor segmentation, transcript counting) still needs to run — this skill is the final export step, not the pipeline itself.

## Inputs

- Sopa-processed SpatialData object stored as a zarr directory (e.g. tuto.zarr)
- Segmented cell boundary polygons (masks already vectorized and merged over patches)
- Gene-count table / cell categories after aggregation and annotation

## Outputs

- tuto.explorer directory containing experiment.xenium
- Pyramidal .ome.tif image file (via sopa.io.explorer.write_image, is_dir=False)
- Cell boundary polygon files compatible with the Xenium Explorer
- Cell category files compatible with the Xenium Explorer
- tuto.explorer/adata.h5ad gene-count AnnData table

## How to apply

Load the processed SpatialData object from the zarr store with spatialdata.read_zarr (e.g. tuto.zarr). First convert the microscopy image into the pyramidal .ome.tif format required by the Xenium Explorer using sopa.io.explorer.write_image (set is_dir=False to emit a single image file). Then run sopa.io.explorer.write (CLI: `sopa explorer write tuto.zarr`) to create the tuto.explorer directory containing experiment.xenium, cell boundary polygons, and cell category files. For time efficiency, split the conversion into a first pass saving only the image (`--mode "+i" --no-save-h5ad`) and a second pass saving boundaries and cell categories after aggregation/annotation (`--mode "-i"`). Finally save the gene-count table as tuto.explorer/adata.h5ad so downstream analyses can use the lightweight AnnData object instead of the full SpatialData object. This works because Sopa's pipeline already converts segmentation masks into polygons merged over patches, removing artefacts, which the converter then serializes into Explorer-compatible geometry.

## Related tools

- **Sopa** (Provides the converter (sopa.io.explorer.write / write_image and the `sopa explorer write` CLI) that transforms the Sopa SpatialData output into Xenium Explorer-compatible files.) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Framework on which the Sopa output is built; the zarr store (read with spatialdata.read_zarr) is the converter's input object.) — https://github.com/scverse/spatialdata
- **Xenium** (Target visualizer: the exported experiment.xenium project loads directly in the Xenium Explorer to display cells, transcripts, and image across technologies such as Xenium, MERSCOPE, CosMX, MACSima, and PhenoCycler.)

## Examples

```
sopa explorer write tuto.zarr  # or: sopa explorer write tuto.zarr --mode "+i" --no-save-h5ad
```

## Evaluation signals

- The tuto.explorer directory exists and contains both experiment.xenium and adata.h5ad.
- The pyramidal image file is a single .ome.tif (is_dir=False) loadable as an Explorer pyramid.
- The exported project (experiment.xenium) opens successfully in the Xenium Explorer and displays cell boundaries and cell categories.
- When using the two-pass split (`--mode "+i"` then `--mode "-i"`), the first pass writes only the image and no h5ad, and the second pass writes boundaries and cell categories.
- adata.h5ad contains the gene-count table matching the SpatialData table, allowing downstream analysis without the full SpatialData object.

## Limitations

- This converter targets the Xenium Explorer specifically; it produces Explorer-compatible files rather than a technology-agnostic visualizer format.
- Split-mode conversion (`--mode "+i"` / `--mode "-i"`) is required for time efficiency on large images; a single-pass run may be slow for millions of cells.
- The converter operates on Sopa's processed output (merged polygons, aggregated counts); it cannot compensate for segmentation artefacts upstream.

## Evidence

- [intro] the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts: "the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts."
- [abstract] Its output includes user-friendly web reports and visualizer files, as well as comprehensive data files for in-depth analysis.: "Its output includes user-friendly web reports and visualizer files, as well as comprehensive data files for in-depth analysis."
- [abstract] a technology-invariant, memory-efficient pipeline with a unified visualizer for all image-based spatial omics: "a technology-invariant, memory-efficient pipeline with a unified visualizer for all image-based spatial omics."
- [other] Sopa incorporates a converter that transforms the pipeline output into input files compatible with the Xenium Explorer: [evidence span withheld — anchor status: not_found]
