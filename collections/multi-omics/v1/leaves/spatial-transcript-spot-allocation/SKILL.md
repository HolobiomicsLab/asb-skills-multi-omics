---
name: spatial-transcript-spot-allocation
description: Use when you have a segmented labels layer (cell shapes) plus a raw transcript
  spot coordinate file from an imaging-based spatial transcriptomics run (RESOLVE,
  MERSCOPE, Xenium, Visium HD) and need a cell-by-gene AnnData count table.
when_to_use_negative:
- Input is already a cell-by-gene feature/count table — transcript allocation only
  makes sense from raw spot coordinates plus shapes
- No segmentation output exists yet — run SPArrOW's `im.segment`/`im.segment_points`
  stage first
- Data is not spatially resolved spot-based transcriptomics (e.g. bulk RNA-seq or
  already-binnedVisium spots)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SPArrOW
  role: 'Provides the allocation stage: sh.vectorize (labels to polygons), tb.allocate
    (spot-to-shape assignment), tb.bin_counts (cell-by-gene binning), tb.add_table_layer
    (attach count matrix)'
  repo: https://github.com/saeyslab/napari-sparrow
- name: napari-sparrow
  role: Napari plugin used to tune allocation/segmentation parameters interactively
    on small crops before running the pipeline
  repo: https://github.com/saeyslab/napari-sparrow
- name: Python
  role: Runtime for SPArrOW, which requires Python 3.11 or newer
  repo: https://github.com/saeyslab/napari-sparrow
- name: uv
  role: Used to create virtual environments and install SPArrOW dependencies
  repo: https://github.com/saeyslab/napari-sparrow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.07.04.601829
    title: 'SPArrOW: a flexible, interactive and scalable pipeline for spatial transcriptomics
      analysis'
  source_content_hash: sha256:c934815e221eecfc90fbef06b565e1f042642310cfb00f55c679bb6c3e397c35
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/saeyslab/napari-SPArrOW
  repo_url: https://github.com/saeyslab/napari-sparrow
  tools:
  - Python
  - SPArrOW
  - napari-sparrow
  - uv
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 9
---

# spatial-transcript-spot-allocation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A SPArrOW pipeline skill that assigns detected transcript spots (e.g. from the RESOLVE mouse liver dataset) to segmented cell shapes via `tb.allocate`, then bins the assignments into a cell-by-gene count matrix via `tb.bin_counts` and attaches it as an AnnData table layer. It bridges the segmentation and transcriptomics-preprocessing stages of the SPArrOW pipeline.

## When to use

Use when you have a segmented labels layer (cell shapes) plus a raw transcript spot coordinate file from an imaging-based spatial transcriptomics run (RESOLVE, MERSCOPE, Xenium, Visium HD) and need a cell-by-gene AnnData count table.

## When NOT to use

- Input is already a cell-by-gene feature/count table — transcript allocation only makes sense from raw spot coordinates plus shapes
- No segmentation output exists yet — run SPArrOW's `im.segment`/`im.segment_points` stage first
- Data is not spatially resolved spot-based transcriptomics (e.g. bulk RNA-seq or already-binnedVisium spots)

## Inputs

- Segmented cell shapes: labels layer or polygons from SPArrOW segmentation output (task_001 output)
- Raw transcript spot coordinate file from the deposited RESOLVE mouse liver dataset
- SPArrOW configuration YAML (e.g. configs/experiment/resolve_liver.yaml for the RESOLVE mouse liver experiment)

## Outputs

- cell_by_gene_table.h5ad: AnnData object with a cell-by-gene count matrix (one row per segmented cell, one column per detected gene)

## How to apply

Load the segmented cell shapes (labels/polygons) from the segmentation output and the raw transcript spot coordinate file from the deposited dataset. Convert the labels layer into polygon shapes with SPArrOW's `sh.vectorize`, because `tb.allocate` operates on shapes rather than a raw labels image. Assign each detected transcript spot to the overlapping cell polygon using SPArrOW's `tb.allocate`, then aggregate the per-cell assignments into per-gene counts with `tb.bin_counts`, producing the cell-by-gene count matrix. Attach the resulting matrix as a table layer with `tb.add_table_layer` and write the AnnData object to disk as `cell_by_gene_table.h5ad`. This stage follows segmentation (`im.segment`) and feeds downstream transcriptomics preprocessing (`tb.preprocess_transcriptomics`) and clustering (`tb.leiden`, `tb.kmeans`) in the SPArrOW pipeline, which can be tuned interactively in the napari plugin on small crops.

## Related tools

- **SPArrOW** (Provides the allocation stage: sh.vectorize (labels to polygons), tb.allocate (spot-to-shape assignment), tb.bin_counts (cell-by-gene binning), tb.add_table_layer (attach count matrix)) — https://github.com/saeyslab/napari-sparrow
- **napari-sparrow** (Napari plugin used to tune allocation/segmentation parameters interactively on small crops before running the pipeline) — https://github.com/saeyslab/napari-sparrow
- **Python** (Runtime for SPArrOW, which requires Python 3.11 or newer) — https://github.com/saeyslab/napari-sparrow
- **uv** (Used to create virtual environments and install SPArrOW dependencies) — https://github.com/saeyslab/napari-sparrow

## Examples

```
import sparrow as sp; sdata.shapes = sp.sh.vectorize(sdata.shapes); sdata = sp.tb.allocate(sdata); sdata = sp.tb.bin_counts(sdata); sp.tb.add_table_layer(sdata).write('cell_by_gene_table.h5ad')
```

## Evaluation signals

- The saved cell_by_gene_table.h5ad loads successfully as an AnnData object
- The count matrix is cell-by-gene: one row per segmented cell, one column per detected gene
- Matrix counts reproduce the counts implied by allocating the input transcripts to the input shapes (allocation/bin_counts invariance)
- The AnnData object contains a table layer (added via tb.add_table_layer) rather than only shapes/points layers

## Limitations

- The package is still under very active development, so APIs (tb.allocate, tb.bin_counts, sh.vectorize) may change
- Allocation quality depends on upstream segmentation; labels must be converted to polygon shapes (sh.vectorize) before tb.allocate can assign spots
- Parameter tuning is recommended on small crops via the napari plugin before full-dataset runs

## Evidence
