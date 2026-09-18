---
name: anndata-table-construction
description: Use when you have segmented cell labels/polygons and raw transcript spot
  coordinates (e.g. from the RESOLVE mouse liver dataset) and need to allocate transcripts
  to cells to produce a cell-by-gene .h5ad count table.
when_to_use_negative:
- Input is already a cell-by-gene count table (e.g. a preprocessed AnnData from tb.preprocess_transcriptomics)
  — proceed to table operations like tb.kmeans or tb.leiden instead of re-allocating
- No segmented cell shapes are available and cell boundaries cannot be derived (e.g.
  sh.create_voronoi_boundaries has not been run) — allocation requires overlapping
  cell polygons
- Data is image-only without detected transcript spot coordinates — this skill allocates
  spots, it does not perform spot detection
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3673
tools:
- name: SPArrOW
  role: Provides the transcript allocation steps tb.allocate (assign spots to cell
    shapes), tb.bin_counts (bin into cell-by-gene counts), tb.add_table_layer (attach
    table to AnnData), and sh.vectorize (labels to polygons)
  repo: https://github.com/saeyslab/napari-sparrow
- name: napari-sparrow
  role: Interactive Napari plugin used to tune SPArrOW pipeline parameters on small
    crops before running allocation
  repo: https://github.com/saeyslab/napari-sparrow
- name: Python
  role: Runtime for the SPArrOW pipeline (requires Python 3.11 or newer, environments
    managed with uv)
merged_aliases:
- anndata-table-manipulation
merged_alias_records:
- alias: anndata-table-manipulation
  slug: anndata-table-manipulation
  cosine_score: 0.8076
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1101/2024.07.04.601829
    title: 'SPArrOW: a flexible, interactive and scalable pipeline for spatial transcriptomics
      analysis'
  source_content_hash: sha256:fdd40db459907170c23b8e590592d4dc5870b0fbdd39f30797e115e46d7a2ada
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# anndata-table-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a cell-by-gene count AnnData table from segmented cell shapes and detected transcript spot coordinates using SPArrOW's shape vectorization and transcript allocation steps (tb.allocate, tb.bin_counts, tb.add_table_layer).

## When to use

Use when you have segmented cell labels/polygons and raw transcript spot coordinates (e.g. from the RESOLVE mouse liver dataset) and need to allocate transcripts to cells to produce a cell-by-gene .h5ad count table.

## When NOT to use

- Input is already a cell-by-gene count table (e.g. a preprocessed AnnData from tb.preprocess_transcriptomics) — proceed to table operations like tb.kmeans or tb.leiden instead of re-allocating
- No segmented cell shapes are available and cell boundaries cannot be derived (e.g. sh.create_voronoi_boundaries has not been run) — allocation requires overlapping cell polygons
- Data is image-only without detected transcript spot coordinates — this skill allocates spots, it does not perform spot detection

## Inputs

- Segmented cell shapes (labels layer / polygons) from upstream segmentation (im.segment)
- Raw transcript spot coordinate file from the deposited spatial transcriptomics dataset (e.g. RESOLVE mouse liver experiment)

## Outputs

- cell_by_gene_table.h5ad — AnnData object containing a cell-by-gene count matrix with one row per segmented cell and one column per detected gene
- Intermediate per-cell transcript assignments produced by tb.allocate

## How to apply

Load the segmented cell shapes (labels layer) from the segmentation output together with the raw transcript spot coordinate file from the deposited dataset (e.g. the RESOLVE mouse liver experiment preconfigured via configs/experiment/resolve_liver.yaml). Convert the labels layer into polygon shapes with SPArrOW's sh.vectorize, then run tb.allocate to assign each transcript spot to the overlapping cell polygon. Aggregate the per-cell assignments into per-cell gene counts with tb.bin_counts, and attach the resulting count matrix as a table layer using tb.add_table_layer. Write the AnnData object to disk as cell_by_gene_table.h5ad; transcript allocation is implemented as two consecutive pipeline steps (tb.allocate followed by tb.bin_counts), so both must be run in order.

## Related tools

- **SPArrOW** (Provides the transcript allocation steps tb.allocate (assign spots to cell shapes), tb.bin_counts (bin into cell-by-gene counts), tb.add_table_layer (attach table to AnnData), and sh.vectorize (labels to polygons)) — https://github.com/saeyslab/napari-sparrow
- **napari-sparrow** (Interactive Napari plugin used to tune SPArrOW pipeline parameters on small crops before running allocation) — https://github.com/saeyslab/napari-sparrow
- **Python** (Runtime for the SPArrOW pipeline (requires Python 3.11 or newer, environments managed with uv))

## Examples

```
uv run sparrow tb.allocate --config configs/experiment/resolve_liver.yaml && uv run sparrow tb.bin_counts --config configs/experiment/resolve_liver.yaml
```

## Evaluation signals

- The saved cell_by_gene_table.h5ad loads successfully with an AnnData reader
- The count matrix has exactly one row per segmented cell and one column per detected gene
- Per-cell gene counts reproduce the counts implied by allocating the input transcript spots to the input shapes (spot-level reconciliation check)
- Each transcript spot is assigned to at most one overlapping cell polygon by tb.allocate (no double counting)
- The AnnData contains a table layer attached via tb.add_table_layer

## Limitations

- The SPArrOW package is still under very active development, so APIs and configs may change
- Allocation quality depends on upstream segmentation; SPArrOW supports parameter tuning via the Napari plugin on small crops before full runs
- Requires Python 3.11 or newer and uv-managed environments as configured in the repository

## Evidence

- [intro] SPArrOW library scope: "SPArrOW: A library for Spatial Transcriptomics Data Analysis."
- [methods] RESOLVE liver experiment preconfigured: "The RESOLVE mouse liver experiment is preconfigured in `configs/experiment/resolve_liver.yaml`"
- [methods] Plugin parameter tuning guidance: "Use the plugin to tune the parameters of `SPArrOW` for the different steps of the pipeline. Tuning can be done on small crops of the image."
- [readme] Active development caveat: "Note: This package is still under very active development."
- [methods] Python version requirement: "The project requires Python 3.11 or newer."
