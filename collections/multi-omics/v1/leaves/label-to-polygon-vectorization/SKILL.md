---
name: label-to-polygon-vectorization
description: Use when you hold a segmentation labels layer from a spatial transcriptomics
  dataset (e.g. RESOLVE mouse liver) and need polygon shapes as the geometric input
  for transcript allocation.
when_to_use_negative:
- Input is already a polygon/shapes layer rather than a labels layer
- You only need image preprocessing or segmentation (use im.tiling_correction, im.enhance_contrast,
  im.segment instead)
- Transcript spots are already assigned to cells and a count table exists (proceed
  to tb.bin_counts / tb.add_table_layer)
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SPArrOW
  role: Provides `sh.vectorize` to convert the labels layer into polygon cell shapes
    consumed by `tb.allocate`
  repo: https://github.com/saeyslab/napari-sparrow
- name: napari-sparrow
  role: Napari plugin distribution of the SPArrOW pipeline steps, usable to inspect/tune
    parameters on image crops
  repo: https://github.com/saeyslab/napari-sparrow
- name: napari
  role: Interactive viewer to inspect labels and vectorized polygon shapes during
    parameter tuning
  repo: https://github.com/saeyslab/napari-sparrow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.07.04.601829
    title: 'SPArrOW: a flexible, interactive and scalable pipeline for spatial transcriptomics
      analysis'
  source_content_hash: sha256:9a77eb0cb09898522e459389a25d56c043da75128395e33f30d642c8396231e1
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
  - napari
  - napari-sparrow
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.25
n_claims: 8
---

# label-to-polygon-vectorization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts a segmented cell labels layer (integer label image) into polygon cell shapes with SPArrOW's `sh.vectorize`, so that transcript spots can be assigned to cells by geometric overlap in the SPArrOW pipeline.

## When to use

Use when you hold a segmentation labels layer from a spatial transcriptomics dataset (e.g. RESOLVE mouse liver) and need polygon shapes as the geometric input for transcript allocation.

## When NOT to use

- Input is already a polygon/shapes layer rather than a labels layer
- You only need image preprocessing or segmentation (use im.tiling_correction, im.enhance_contrast, im.segment instead)
- Transcript spots are already assigned to cells and a count table exists (proceed to tb.bin_counts / tb.add_table_layer)

## Inputs

- Segmented cell labels layer (integer label image, e.g. from SPArrOW im.segment)
- Raw transcript spot coordinate file from the deposited spatial transcriptomics dataset (e.g. RESOLVE mouse liver)

## Outputs

- Polygon shapes (one per segmented cell) suitable for SPArrOW tb.allocate

## How to apply

Load the segmented cell shapes (labels layer) produced by SPArrOW's segmentation stage (`im.segment` / `im.segment_points`) together with the raw transcript spot coordinates from the deposited dataset. Convert the labels layer into polygon shapes using SPArrOW's `sh.vectorize`, since `tb.allocate` assigns transcripts to cell polygons rather than label images. Vectorize before running the two-step allocation stage (`tb.allocate` followed by `tb.bin_counts`), which bins the assigned transcripts into a cell-by-gene count table that is attached with `tb.add_table_layer` and written as cell_by_gene_table.h5ad. Each polygon should correspond to one segmented cell so the downstream count matrix has one row per segmented cell.

## Related tools

- **SPArrOW** (Provides `sh.vectorize` to convert the labels layer into polygon cell shapes consumed by `tb.allocate`) — https://github.com/saeyslab/napari-sparrow
- **napari-sparrow** (Napari plugin distribution of the SPArrOW pipeline steps, usable to inspect/tune parameters on image crops) — https://github.com/saeyslab/napari-sparrow
- **napari** (Interactive viewer to inspect labels and vectorized polygon shapes during parameter tuning) — https://github.com/saeyslab/napari-sparrow

## Examples

```
python sparrow_pipeline.py run sh.vectorize tb.allocate tb.bin_counts tb.add_table_layer --config configs/experiment/resolve_liver.yaml
```

## Evaluation signals

- Number of output polygons equals the number of distinct labels in the input labels layer
- Each polygon overlaps the pixel footprint of its source label (geometric consistency check)
- Downstream `tb.allocate` assigns every detected transcript to a cell polygon and `tb.bin_counts` yields a cell-by-gene matrix with one row per segmented cell
- Saved cell_by_gene_table.h5ad loads successfully in AnnData and reproduces counts implied by allocating the input transcripts to the vectorized shapes

## Limitations

- The SPArrOW package is still under very active development, so function names/behavior may change
- Vectorization quality depends on the fidelity of the upstream segmentation labels layer; errors in `im.segment` propagate into the polygons

## Evidence

- [methods] workflow step: vectorize labels to polygons: [evidence span withheld — anchor status: not_found]
