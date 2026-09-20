---
name: pyramidal-ome-tif-image-export
description: Use when a processed SpatialData object (e.g. tuto.zarr) must be opened
  in the Xenium Explorer and the microscopy image still needs conversion into the
  Explorer's pyramidal .ome.tif format.
when_to_use_negative:
- The image has already been converted to the Explorer's pyramidal .ome.tif and only
  boundaries/cell categories remain (use the second-pass mode '-i' instead of re-exporting
  the image)
- The goal is downstream numerical analysis rather than visualization — save the gene-count
  table as tuto.explorer/adata.h5ad and use the AnnData object instead of the full
  SpatialData object
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Sopa
  role: 'Provides sopa.io.explorer.write_image and sopa.io.explorer.write (CLI: sopa
    explorer write) to produce the Explorer-compatible output files including the
    pyramidal .ome.tif'
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Stores the processed pipeline output (read with spatialdata.read_zarr) whose
    image is converted to .ome.tif
  repo: https://github.com/scverse/spatialdata
- name: Xenium
  role: 'Target visualizer: the pyramidal .ome.tif plus experiment.xenium and boundary
    files form a project loadable in the Xenium Explorer'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:18cab0fee7213a25273ed2d59ccfc79d8f7223b08266b710ef6bb7e86e0d82f1
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# pyramidal-ome-tif-image-export

## Summary

Converts the microscopy image stored in a Sopa/SpatialData object into the pyramidal .ome.tif format required by the Xenium Explorer, so pipeline output from any image-based spatial omics technology (Xenium, MERSCOPE, CosMX, MACSima, PhenoCycler) can be visualized in a shared visualizer.

## When to use

Use when a processed SpatialData object (e.g. tuto.zarr) must be opened in the Xenium Explorer and the microscopy image still needs conversion into the Explorer's pyramidal .ome.tif format.

## When NOT to use

- The image has already been converted to the Explorer's pyramidal .ome.tif and only boundaries/cell categories remain (use the second-pass mode '-i' instead of re-exporting the image)
- The goal is downstream numerical analysis rather than visualization — save the gene-count table as tuto.explorer/adata.h5ad and use the AnnData object instead of the full SpatialData object

## Inputs

- Processed SpatialData object stored as a .zarr store (e.g. tuto.zarr)
- Microscopy image channels contained in the SpatialData object

## Outputs

- Pyramidal OME-TIFF image file (e.g. tuto.explorer/image.ome.tif) with is_dir=False
- Files contributed to the tuto.explorer directory that loads as a Xenium Explorer project (together with experiment.xenium, cell boundary polygons, cell category files)

## How to apply

Load the processed SpatialData object from its .zarr directory with spatialdata.read_zarr, then call sopa.io.explorer.write_image to convert the microscopy image into the pyramidal .ome.tif required by the Xenium Explorer; set is_dir=False to emit a single image file rather than a directory of tiles. This export can be run standalone as a first pass (CLI mode '+i' with --no-save-h5ad) so it runs in parallel with segmentation/aggregation, while cell boundary polygons and cell category files are written in a second pass (mode '-i') after annotation. The resulting image file is placed in the tuto.explorer output directory alongside experiment.xenium, cell boundary polygons, and cell category files, and the gene-count table is saved as adata.h5ad. Rationale: the Explorer expects optimized, pyramidal imagery, and Sopa's converter makes the Explorer serve as a unified visualizer for any technology whose data Sopa can read.

## Related tools

- **Sopa** (Provides sopa.io.explorer.write_image and sopa.io.explorer.write (CLI: sopa explorer write) to produce the Explorer-compatible output files including the pyramidal .ome.tif) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Stores the processed pipeline output (read with spatialdata.read_zarr) whose image is converted to .ome.tif) — https://github.com/scverse/spatialdata
- **Xenium** (Target visualizer: the pyramidal .ome.tif plus experiment.xenium and boundary files form a project loadable in the Xenium Explorer)

## Examples

```
sdata = spatialdata.read_zarr("tuto.zarr"); sopa.io.explorer.write_image(sdata, "tuto.explorer/image.ome.tif", is_dir=False)
```

## Evaluation signals

- The tuto.explorer directory contains experiment.xenium and the exported pyramidal .ome.tif image file
- The gene-count table tuto.explorer/adata.h5ad exists alongside the image export
- The tuto.explorer project loads successfully in the Xenium Explorer with the image rendered
- When using the two-pass split, the image-only first pass ran with --mode "+i" --no-save-h5ad and boundaries/categories followed with --mode "-i"

## Limitations

- Time efficiency motivates splitting the conversion: saving the image alone first and boundaries/cell categories only after aggregation/annotation; doing everything in one pass is slower
- is_dir must be set correctly — False for a single image file; the wrong setting changes the output layout
- Sopa's software version numbers and dataset accessions are not reported in the discussion section, so version pinning must be established separately

## Evidence

- [other] Pyramidal ome-tif conversion for Xenium Explorer: [evidence span withheld — anchor status: not_found]
- [other] Two-pass split for time efficiency: [evidence span withheld — anchor status: not_found]
- [abstract] Unified visualizer output: "Its output includes user-friendly web reports and visualizer files, as well as comprehensive data files for in-depth analysis."
