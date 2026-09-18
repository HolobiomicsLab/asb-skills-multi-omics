---
name: kmeans-clustering-of-spatial-transcriptomics-cells
description: Use when you have an allocated, preprocessed cell-by-gene AnnData table
  from a spatial transcriptomics pipeline (e.g. SPArrOW tb.allocate followed by tb.preprocess_transcriptomics)
  and need unsupervised cell clustering.
when_to_use_negative:
- Input is raw imaging data or transcript locations rather than an allocated, preprocessed
  cell-by-gene table — run image processing (im.segment, im.tiling_correction) and
  tb.allocate / tb.preprocess_transcriptomics first
- You need graph/community-based clustering with resolution tuning — use the SPArrOW
  Leiden operation (tb.leiden) instead of, or in addition to, k-means
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SPArrOW
  role: Provides the tb.kmeans table operation that assigns k-means cluster labels
    to cells in the cell-by-gene table
  repo: https://github.com/saeyslab/napari-sparrow
- name: Python
  role: Runtime environment required by SPArrOW (Python 3.11 or newer) in which tb.kmeans
    is executed
- name: napari-sparrow
  role: Napari plugin used to tune the parameters of SPArrOW pipeline steps, including
    clustering, on small image crops
  repo: https://github.com/saeyslab/napari-sparrow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.07.04.601829
    title: 'SPArrOW: a flexible, interactive and scalable pipeline for spatial transcriptomics
      analysis'
  source_content_hash: sha256:9bcd4fbc8e116b1201c0eabd19458817bf93330793d18d0baf1608819deb0818
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.0
n_claims: 8
---

# kmeans-clustering-of-spatial-transcriptomics-cells

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assign k-means cluster labels to cells in a preprocessed spatial transcriptomics cell-by-gene table using the SPArrOW table operation tb.kmeans, storing the result as an alternative clustering alongside the Leiden clustering (tb.leiden).

## When to use

Use when you have an allocated, preprocessed cell-by-gene AnnData table from a spatial transcriptomics pipeline (e.g. SPArrOW tb.allocate followed by tb.preprocess_transcriptomics) and need unsupervised cell clustering.

## When NOT to use

- Input is raw imaging data or transcript locations rather than an allocated, preprocessed cell-by-gene table — run image processing (im.segment, im.tiling_correction) and tb.allocate / tb.preprocess_transcriptomics first
- You need graph/community-based clustering with resolution tuning — use the SPArrOW Leiden operation (tb.leiden) instead of, or in addition to, k-means

## Inputs

- Preprocessed cell-by-gene AnnData table from tb.preprocess_transcriptomics (following tb.allocate / tb.bin_counts transcript allocation)
- Number of k-means clusters (k) and related clustering parameters

## Outputs

- AnnData object annotated with a k-means cluster assignment for every cell, stored as an alternative clustering alongside tb.leiden
- Serialized .h5ad output file containing the k-means clustering
- Configuration .yaml file capturing the k-means step parameters, saved in the output directory

## How to apply

Load the preprocessed cell-by-gene AnnData table produced by the transcriptomics preprocessing step (tb.preprocess_transcriptomics), which follows SPArrOW's transcript allocation step (tb.allocate / tb.bin_counts). Run the SPArrOW table operation tb.kmeans on the table to assign k-means cluster labels to every cell. Store the k-means assignments as an alternative clustering alongside the Leiden clustering (tb.leiden) computed from the same input, so the two unsupervised clusterings can be compared. SPArrOW is configuration-driven: after each pipeline step a corresponding configuration .yaml file is saved in the chosen output directory, so record the k-means parameters there. Save the annotated AnnData object with the k-means clustering to an .h5ad output file. If cluster parameters need adjustment, the napari-sparrow plugin can be used to tune SPArrOW step parameters on small crops of the image before running at scale.

## Related tools

- **SPArrOW** (Provides the tb.kmeans table operation that assigns k-means cluster labels to cells in the cell-by-gene table) — https://github.com/saeyslab/napari-sparrow
- **Python** (Runtime environment required by SPArrOW (Python 3.11 or newer) in which tb.kmeans is executed)
- **napari-sparrow** (Napari plugin used to tune the parameters of SPArrOW pipeline steps, including clustering, on small image crops) — https://github.com/saeyslab/napari-sparrow

## Evaluation signals

- The output .h5ad file exists and can be loaded as an AnnData object
- A k-means cluster assignment column exists for every cell in the table (no null assignments)
- The k-means clustering is stored as an alternative clustering alongside the Leiden (tb.leiden) clustering computed from the same input
- A corresponding configuration .yaml file for the clustering step was written to the chosen output directory
- Cluster labels are valid categorical identifiers covering all cells in the cell-by-gene matrix

## Limitations

- The SPArrOW package is still under very active development, so the tb.kmeans API and configuration options may change
- k-means is one of two clustering options in SPArrOW (tb.leiden and tb.kmeans); choosing k depends on the biology and should ideally be tuned via the Napari plugin on small crops of the image

## Evidence
