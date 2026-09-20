---
name: spatialdata-zarr-handling
description: Use when raw output from any image-based spatial omics technology (Xenium,
  MERSCOPE, CosMX, MACSima, PhenoCycler, or toy datasets) must be converted into a
  persistent SpatialData .zarr store that downstream Sopa steps and the Xenium Explorer
  converter can read.
when_to_use_negative:
- The data are already segmented and only cell-level counts are needed — reading an
  existing .h5ad or count matrix is simpler than round-tripping through a .zarr store
- Working outside the Sopa/SpatialData ecosystem where no SpatialData reader exists
  and the zarr directory cannot be interpreted
edam_operation: http://edamontology.org/operation_2409
tools:
- name: SpatialData
  role: Universal framework providing the unified data representation and read_zarr/write_zarr
    I/O for the .zarr store
  repo: https://github.com/scverse/spatialdata
- name: Sopa
  role: Converts any technology-specific input into a SpatialData object (sopa convert)
    and consumes the .zarr store in every pipeline step, including the Xenium Explorer
    export
  repo: https://github.com/gustaveroussy/sopa
- name: Proseg
  role: Alternative segmentation tool that also emits a SpatialData-format .zarr directory
    with cell_boundaries shapes and an AnnData table, interchangeable with Sopa's
    store
  repo: https://github.com/dcjones/proseg
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:f6d9cc2553a9770ef72c61d96dfb27e386ff34697b7591f50359c48e16126b4d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/spatialdata
  tools:
  - Proseg
  - Sopa
  - SpatialData
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# spatialdata-zarr-handling

## Summary

Convert and manage image-based spatial omics data as a unified SpatialData object stored in a .zarr directory, so all Sopa tasks (segmentation, aggregation, annotation, spatial analysis, and Explorer export) operate on one consistent store.

## When to use

Use when raw output from any image-based spatial omics technology (Xenium, MERSCOPE, CosMX, MACSima, PhenoCycler, or toy datasets) must be converted into a persistent SpatialData .zarr store that downstream Sopa steps and the Xenium Explorer converter can read.

## When NOT to use

- The data are already segmented and only cell-level counts are needed — reading an existing .h5ad or count matrix is simpler than round-tripping through a .zarr store
- Working outside the Sopa/SpatialData ecosystem where no SpatialData reader exists and the zarr directory cannot be interpreted

## Inputs

- Raw spatial omics technology output (e.g. Xenium, MERSCOPE, CosMX transcript/molecule files, microscopy images)
- Generated toy dataset via sopa.io.toy_dataset
- Existing .zarr SpatialData directories (e.g. tuto.zarr, proseg-output.zarr)

## Outputs

- SpatialData .zarr store (e.g. tuto.zarr) containing images, points['transcripts'], shapes['cell_boundaries'], and tables['table']
- Extracted AnnData .h5ad file (e.g. adata.h5ad) for downstream analyses

## How to apply

First convert the technology-specific input into a SpatialData object with `sopa convert`, writing it to a .zarr directory (e.g. tuto.zarr) via the --sdata-path argument. Read the store back with spatialdata.read_zarr; within the object, transcripts live in sdata.points['transcripts'], cell polygons in sdata.shapes['cell_boundaries'], and cell-level AnnData (obs, var, sparse cell-by-gene matrix in X, centroids in obsm['spatial']) in sdata.tables['table']. All subsequent Sopa steps (patchify, segmentation, resolve, aggregate, Explorer export) take the .zarr path as input, so keep the store as the single source of truth throughout the pipeline. When downstream tools only accept AnnData, extract the table and write it as .h5ad (e.g. sdata.tables['table'].write_h5ad('adata.h5ad')). The rationale is Sopa's design goal: a universal, technology-invariant data representation built on the SpatialData framework that unifies segmentation, aggregation, annotation, and geometric/spatial analysis.

## Related tools

- **SpatialData** (Universal framework providing the unified data representation and read_zarr/write_zarr I/O for the .zarr store) — https://github.com/scverse/spatialdata
- **Sopa** (Converts any technology-specific input into a SpatialData object (sopa convert) and consumes the .zarr store in every pipeline step, including the Xenium Explorer export) — https://github.com/gustaveroussy/sopa
- **Proseg** (Alternative segmentation tool that also emits a SpatialData-format .zarr directory with cell_boundaries shapes and an AnnData table, interchangeable with Sopa's store) — https://github.com/dcjones/proseg

## Examples

```
sopa convert . --sdata-path tuto.zarr --technology toy_dataset
```

## Evaluation signals

- The .zarr path loads without error via spatialdata.read_zarr and exposes points['transcripts'], shapes['cell_boundaries'], and tables['table']
- tables['table'] contains cell metadata in obs, gene metadata in var, a sparse cell-by-gene count matrix in X, and centroids in obsm['spatial']
- Downstream Sopa CLI commands (sopa patchify, sopa segmentation cellpose, sopa resolve, sopa explorer write) all accept the same .zarr path
- The exported adata.h5ad opens in scanpy/zellkonverter and matches the cell-by-gene counts in the SpatialData table

## Limitations

- Transcript points can use significant disk space; proseg offers --exclude-spatialdata-transcripts to omit them from the zarr output
- No software version numbers or dataset accessions for the zarr stores are reported in the article's discussion section

## Evidence

- [abstract] SpatialData universal framework: "Built upon the universal SpatialData framework, Sopa optimizes tasks like segmentation, transcript/channel aggregation, annotation, and geometric/spatial analysis."
- [intro] SpatialData conversion of any input: "Sopa converts any data into a SpatialData object, on which all of the six following tasks are performed."
- [methods] Sopa built on SpatialData repo: "Built on top of [SpatialData](https://github.com/scverse/spatialdata), Sopa enables processing"
- [readme] Proseg spatialdata zarr read pattern: [evidence span withheld — anchor status: not_found]
- [readme] Proseg SpatialData object contents: [evidence span withheld — anchor status: not_found]
