---
name: gene-count-anndata-export
description: Use when the Sopa pipeline has finished aggregation (transcripts counted
  per cell) and you need a portable cell-by-gene matrix for scanpy/AnnData-based downstream
  analysis without reloading the SpatialData object.
when_to_use_negative:
- The pipeline has not yet run aggregation, so no per-cell transcript counts exist
  to export
- You still need image, transcript, and shape elements for visualization or analysis
  — the h5ad contains only the cell-by-gene matrix and cell metadata, not the full
  SpatialData payload
- The target visualizer requires only the Explorer files (experiment.xenium, boundaries);
  the h5ad export is for downstream analysis, not for loading the project in Xenium
  Explorer
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Sopa
  role: Provides sopa.io.explorer.write and the `sopa explorer write` CLI with --mode
    and --no-save-h5ad flags to emit the adata.h5ad gene-count export alongside Explorer
    files
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Source of the processed zarr store whose aggregated gene-count table is converted
    into the AnnData h5ad export
  repo: https://github.com/scverse/spatialdata
- name: Xenium
  role: Target visualizer format; the h5ad is saved in the tuto.explorer directory
    alongside the Explorer-compatible files
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:446d95cc540b3ea01638260479b73939f28119950d0a63bfb68d1d665aab4567
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - Sopa
  - SpatialData
  - Xenium
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# gene-count-anndata-export

## Summary

Exports the per-cell gene-count table produced by Sopa's aggregation step as an AnnData .h5ad file (e.g. tuto.explorer/adata.h5ad) so downstream analyses can load a lightweight AnnData object instead of the full SpatialData zarr store.

## When to use

Use when the Sopa pipeline has finished aggregation (transcripts counted per cell) and you need a portable cell-by-gene matrix for scanpy/AnnData-based downstream analysis without reloading the SpatialData object.

## When NOT to use

- The pipeline has not yet run aggregation, so no per-cell transcript counts exist to export
- You still need image, transcript, and shape elements for visualization or analysis — the h5ad contains only the cell-by-gene matrix and cell metadata, not the full SpatialData payload
- The target visualizer requires only the Explorer files (experiment.xenium, boundaries); the h5ad export is for downstream analysis, not for loading the project in Xenium Explorer

## Inputs

- Processed SpatialData object stored as a zarr directory (e.g. tuto.zarr)
- Aggregated cell-by-gene transcript count table within the SpatialData table slot

## Outputs

- tuto.explorer/adata.h5ad — AnnData file containing the gene-count table for downstream analysis

## How to apply

After segmentation and aggregation, run the Xenium Explorer conversion with sopa.io.explorer.write (CLI: `sopa explorer write tuto.zarr`) and request that the gene-count table be saved as adata.h5ad inside the tuto.explorer output directory. For time efficiency, split the conversion into two passes: a first pass with `--mode "+i" --no-save-h5ad` that writes only the pyramidal .ome.tif image via sopa.io.explorer.write_image, and a second pass with `--mode "-i"` that writes cell boundary polygons and cell categories after aggregation/annotation. The rationale is that the full SpatialData object is memory-heavy, whereas the saved AnnData contains the sparse cell-by-gene counts and cell metadata needed for annotation, spatial analysis, and reuse in scanpy-style workflows. Verify the export by loading the h5ad and confirming it matches the cells in the SpatialData table.

## Related tools

- **Sopa** (Provides sopa.io.explorer.write and the `sopa explorer write` CLI with --mode and --no-save-h5ad flags to emit the adata.h5ad gene-count export alongside Explorer files) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Source of the processed zarr store whose aggregated gene-count table is converted into the AnnData h5ad export) — https://github.com/scverse/spatialdata
- **Xenium** (Target visualizer format; the h5ad is saved in the tuto.explorer directory alongside the Explorer-compatible files)

## Examples

```
sopa explorer write tuto.zarr --mode "-i"  # second pass; saves tuto.explorer/adata.h5ad with the gene-count table
```

## Evaluation signals

- tuto.explorer directory contains both experiment.xenium and adata.h5ad after the two-pass conversion
- The h5ad loads with an AnnData reader and its number of cells matches the cell count in the SpatialData table
- The --mode "+i" --no-save-h5ad first pass produced the image but no h5ad; the "-i" second pass produced the h5ad
- The project still opens in the Xenium Explorer after the split conversion, confirming the export did not corrupt Explorer files

## Limitations

- The h5ad holds only the gene-count table and cell metadata; analyses needing images, transcripts, or polygons must go back to the SpatialData zarr
- Splitting the export into image-only and boundary-only passes adds orchestration complexity; running a single full conversion is simpler but slower
- Future segmentation or annotation methods must be published and validated before they can change what ends up in the exported table

## Evidence

- [methods] Two-pass Explorer conversion with h5ad export: [evidence span withheld — anchor status: not_found]
- [methods] h5ad purpose: [evidence span withheld — anchor status: not_found]
- [abstract] Sopa outputs include comprehensive data files: "Its output includes user-friendly web reports and visualizer files, as well as comprehensive data files for in-depth analysis."
