---
name: spatial-omics-data-loading
description: Use when raw spatial-omics outputs from Xenium, MERSCOPE or CosMX (transcript
  tables, images) must be converted into a SpatialData object before segmentation,
  aggregation, annotation or spatial analysis with Sopa.
when_to_use_negative:
- Input is already a SpatialData object produced by a prior Sopa step — reload not
  needed
- The data are already-segmented or from a non-image-based technology (e.g. bulk or
  sequencing-based spatial data); Sopa targets image-based spatial omics only
- Only single-cell expression matrices without images or transcript coordinates are
  available
edam_operation: http://edamontology.org/operation_2409
tools:
- name: Sopa
  role: Provides the technology-invariant sopa.io readers (sopa.io.merscope, sopa.io.xenium,
    sopa.io.toy_dataset) that convert raw spatial-omics outputs into SpatialData objects
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Universal data framework on which Sopa builds; the loaded SpatialData object
    is the unified representation for all pipeline tasks
  repo: https://github.com/scverse/spatialdata
- name: uv
  role: Used to install Sopa in editable/dev mode with 'uv sync --all-extras --dev'
    before loading data
  repo: https://github.com/astral-sh/uv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:3bf6773ec84b37fe7776159e3a30eae2a08d6c2682aa856b29bb86aa0518cc86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - MERSCOPE
  - Sopa
  - SpatialData
  - uv
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# spatial-omics-data-loading

## Summary

Load image-based spatial omics data (Xenium, MERSCOPE, CosMX, or synthetic toy data) into the unified SpatialData framework using Sopa's technology-invariant sopa.io readers, the mandatory first step of every Sopa pipeline task.

## When to use

Use when raw spatial-omics outputs from Xenium, MERSCOPE or CosMX (transcript tables, images) must be converted into a SpatialData object before segmentation, aggregation, annotation or spatial analysis with Sopa.

## When NOT to use

- Input is already a SpatialData object produced by a prior Sopa step — reload not needed
- The data are already-segmented or from a non-image-based technology (e.g. bulk or sequencing-based spatial data); Sopa targets image-based spatial omics only
- Only single-cell expression matrices without images or transcript coordinates are available

## Inputs

- Xenium instrument output directory (transcripts, images)
- MERSCOPE output directory (transcripts, images)
- CosMX output directory (transcripts, images)
- No input (synthetic data) for sopa.io.toy_dataset

## Outputs

- SpatialData object wrapping images, transcript tables and metadata
- On-disk SpatialData-consistent store consumable by subsequent Sopa CLI/API steps

## How to apply

Instantiate Sopa in a development environment (clone the repository and run 'uv sync --all-extras --dev'), then call the technology-specific reader: sopa.io.merscope or sopa.io.xenium to wrap proprietary instrument outputs into a SpatialData object, or sopa.io.toy_dataset to generate a synthetic dataset for testing. The resulting SpatialData object (built on the scverse/spatialdata framework) is the single representation on which all six Sopa tasks — segmentation, patching, transcript/channel aggregation, annotation, geometric and spatial analysis — operate, so downstream steps never touch the raw instrument files again. For testing or benchmarking (e.g. RAM/time comparisons of Cellpose or Baysor segmentation with and without patching), the toy dataset provides a reproducible, technology-invariant stand-in. Optionally set sopa.settings.gene_exclude_pattern before downstream segmentation to filter gene names during loading-adjacent steps.

## Related tools

- **Sopa** (Provides the technology-invariant sopa.io readers (sopa.io.merscope, sopa.io.xenium, sopa.io.toy_dataset) that convert raw spatial-omics outputs into SpatialData objects) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Universal data framework on which Sopa builds; the loaded SpatialData object is the unified representation for all pipeline tasks) — https://github.com/scverse/spatialdata
- **uv** (Used to install Sopa in editable/dev mode with 'uv sync --all-extras --dev' before loading data) — https://github.com/astral-sh/uv

## Examples

```
import sopa; sdata = sopa.io.toy_dataset(); sdata_x = sopa.io.xenium('/path/to/xenium/output')
```

## Evaluation signals

- The returned object is a valid SpatialData object (spatialdata.SpatialData) exposing image and transcript layers for the chosen technology
- For sopa.io.toy_dataset, a generated synthetic dataset is produced with a total runtime of a few minutes as documented
- Downstream Sopa steps (e.g. 'sopa patchify image ...') can run directly on the loaded SpatialData object without re-parsing raw files
- Reading a Xenium or MERSCOPE crop yields transcripts and images consistent with the source instrument output (technology-invariant representation)

## Limitations

- Sopa converts any data into a SpatialData object but targets image-based spatial omics technologies (Xenium, MERSCOPE, CosMX, MACSima, PhenoCycler); other data types are not covered by the documented readers
- The loaded object is an intermediate representation; no segmentation or annotation occurs at this step, so a successful load does not validate downstream results
- No software version numbers for Sopa or dependencies are reported, so reproducibility of the loading environment must be pinned separately

## Evidence

- [intro] Sopa converts any data into a SpatialData object: "Sopa converts any data into a SpatialData object, on which all of the six following tasks are performed."
- [abstract] Built upon the universal SpatialData framework: "Built upon the universal SpatialData framework, Sopa optimizes tasks like segmentation, transcript/channel aggregation, annotation, and geometric/spatial analysis."
- [other] sopa.io readers for MERSCOPE and Xenium crops: [evidence span withheld — anchor status: not_found]
- [abstract] diverse spatial omics technologies: "diverse spatial omics technologies, such as Xenium, MERSCOPE, CosMX in spatial-transcriptomics, and MACSima and PhenoCycler in multiplex imaging"
