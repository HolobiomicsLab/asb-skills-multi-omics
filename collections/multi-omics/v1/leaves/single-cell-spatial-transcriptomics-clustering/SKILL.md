---
name: single-cell-spatial-transcriptomics-clustering
description: Use when a preprocessed cell-by-gene AnnData table from spatial transcriptomics
  allocation/preprocessing (e.g. tb.preprocess_transcriptomics) is available and cells
  need unsupervised cluster labels for downstream spatial analysis.
when_to_use_negative:
- Input is raw image or transcript data that has not yet gone through segmentation,
  transcript allocation (tb.allocate/tb.bin_counts), and transcriptomics preprocessing
  — clustering requires the preprocessed cell-by-gene table
- The task is gene scoring (tb.score_genes), neighborhood enrichment (tb.nhood_enrichment),
  or other downstream table operations rather than assigning cell cluster labels
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: SPArrOW
  role: Provides the tb.kmeans and tb.leiden table operations used to cluster cells
    of the allocated cell-by-gene table
  repo: https://github.com/saeyslab/napari-sparrow
- name: napari-sparrow
  role: Napari plugin used to interactively tune SPArrOW pipeline parameters (including
    clustering) on small crops of the image
  repo: https://github.com/saeyslab/napari-sparrow
- name: Python
  role: Runtime for the SPArrOW pipeline (requires Python 3.11 or newer, installed
    via uv)
- name: napari
  role: Host application in which SPArrOW runs as a plugin for parameter tuning
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.07.04.601829
    title: 'SPArrOW: a flexible, interactive and scalable pipeline for spatial transcriptomics
      analysis'
  source_content_hash: sha256:99c8579dc5d4127966b170d20329a91e4c749be8e0c6f29447e6e763b5be8530
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.125
n_claims: 8
---

# single-cell-spatial-transcriptomics-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns cluster labels to single cells in a spatial transcriptomics experiment by running k-means (tb.kmeans) and Leiden (tb.leiden) clustering on a preprocessed cell-by-gene AnnData table within the SPArrOW pipeline, and stores the assignments in an .h5ad output.

## When to use

Use when a preprocessed cell-by-gene AnnData table from spatial transcriptomics allocation/preprocessing (e.g. tb.preprocess_transcriptomics) is available and cells need unsupervised cluster labels for downstream spatial analysis.

## When NOT to use

- Input is raw image or transcript data that has not yet gone through segmentation, transcript allocation (tb.allocate/tb.bin_counts), and transcriptomics preprocessing — clustering requires the preprocessed cell-by-gene table
- The task is gene scoring (tb.score_genes), neighborhood enrichment (tb.nhood_enrichment), or other downstream table operations rather than assigning cell cluster labels

## Inputs

- Preprocessed cell-by-gene AnnData table (output of tb.preprocess_transcriptomics in the SPArrOW pipeline)
- Number of clusters k for k-means (and Leiden resolution) as pipeline configuration parameters

## Outputs

- AnnData object annotated with k-means cluster assignment column for every cell (stored alongside Leiden clustering)
- Output .h5ad file containing the clustered cell-by-gene table
- Corresponding configuration .yaml file saved in the chosen output directory

## How to apply

Load the cell-by-gene AnnData table produced by SPArrOW's transcript allocation (tb.allocate/tb.bin_counts) and preprocessing step (tb.preprocess_transcriptomics). Run the SPArrOW table operation tb.kmeans on the table to assign k-means cluster labels to each cell, and compute the Leiden clustering (tb.leiden) from the same input for comparison. Store the k-means assignments as an alternative clustering alongside the Leiden clustering in the AnnData object rather than overwriting existing labels. Save the annotated AnnData object with both clusterings to an .h5ad output file, following SPArrOW's convention that each pipeline step saves a corresponding configuration .yaml file in the chosen output directory. Parameters can be tuned interactively with the napari-sparrow plugin on small crops before running the full pipeline.

## Related tools

- **SPArrOW** (Provides the tb.kmeans and tb.leiden table operations used to cluster cells of the allocated cell-by-gene table) — https://github.com/saeyslab/napari-sparrow
- **napari-sparrow** (Napari plugin used to interactively tune SPArrOW pipeline parameters (including clustering) on small crops of the image) — https://github.com/saeyslab/napari-sparrow
- **Python** (Runtime for the SPArrOW pipeline (requires Python 3.11 or newer, installed via uv))
- **napari** (Host application in which SPArrOW runs as a plugin for parameter tuning)

## Examples

```
uv run sparrow table kmeans --input preprocessed.h5ad --output clustered.h5ad && uv run sparrow table leiden --input preprocessed.h5ad
```

## Evaluation signals

- The output .h5ad file exists and is a valid AnnData object
- Every cell in the table has a non-null k-means cluster assignment column
- The k-means clustering is stored as an alternative clustering and does not overwrite the Leiden (tb.leiden) labels computed from the same input
- A corresponding configuration .yaml file for the clustering step is saved in the chosen output directory
- The number of distinct k-means labels matches the configured k

## Limitations

- SPArrOW is still under very active development, so interfaces and defaults may change
- Clustering operates on the output of upstream steps (allocation and preprocessing); errors in segmentation or transcript allocation propagate into cluster labels
- The package requires Python 3.11 or newer and uses uv for environment management, so older environments are unsupported

## Evidence
