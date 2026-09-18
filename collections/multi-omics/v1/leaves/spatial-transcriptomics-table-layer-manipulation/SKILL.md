---
name: spatial-transcriptomics-table-layer-manipulation
description: Use when a preprocessed cell-by-gene expression table (AnnData .h5ad)
  from a spatial transcriptomics pipeline needs downstream table-layer operations
  such as clustering (Leiden/KMeans), gene scoring, or neighborhood enrichment.
when_to_use_negative:
- Input is raw image or transcript-level data before allocation — use the image layer
  (im.*) or allocation step (tb.allocate / tb.bin_counts) first
- The table has not been preprocessed (no tb.preprocess_transcriptomics output); clustering
  on unnormalized counts is out of scope
- Proteomics-based analysis is the goal — use tb.preprocess_proteomics rather than
  the transcriptomics table clustering workflow
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SPArrOW
  role: Provides the tb.* table-layer operations, including tb.leiden and tb.kmeans
    clustering on the cell-by-gene AnnData table
  repo: https://github.com/saeyslab/napari-sparrow
- name: Python
  role: Execution environment (Python 3.11 or newer) in which the sparrow.tb module
    and AnnData objects are used
- name: AnnData
  role: In-memory format for the cell-by-gene table; clustering labels are written
    into the .obs column
- name: uv
  role: Creates virtual environments and installs dependencies to run the SPArrOW
    pipeline
- name: Napari
  role: Plugin interface used to tune the parameters of SPArrOW steps, including table-layer
    steps, on small image crops
- name: Hydra
  role: Runs SPArrOW experiments from the CLI, including the clustering configuration
    step
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.07.04.601829
    title: 'SPArrOW: a flexible, interactive and scalable pipeline for spatial transcriptomics
      analysis'
  source_content_hash: sha256:47545ea71e0af07efbc4290f57bc2a92ef5ce532bc291a0e54d6e9753b38a8d4
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
  - AnnData
  - Hydra
  - Napari
  - Python
  - SPArrOW
  - uv
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.36363636363636365
n_claims: 11
---

# spatial-transcriptomics-table-layer-manipulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

This skill covers SPArrOW's table-layer operations (the `tb.*` tools in the `sparrow.tb` module) that operate on the cell-by-gene AnnData table after transcriptomics preprocessing, including Leiden graph-based clustering to assign cluster labels to cells.

## When to use

Use when a preprocessed cell-by-gene expression table (AnnData .h5ad) from a spatial transcriptomics pipeline needs downstream table-layer operations such as clustering (Leiden/KMeans), gene scoring, or neighborhood enrichment.

## When NOT to use

- Input is raw image or transcript-level data before allocation — use the image layer (im.*) or allocation step (tb.allocate / tb.bin_counts) first
- The table has not been preprocessed (no tb.preprocess_transcriptomics output); clustering on unnormalized counts is out of scope
- Proteomics-based analysis is the goal — use tb.preprocess_proteomics rather than the transcriptomics table clustering workflow

## Inputs

- Preprocessed cell-by-gene AnnData table (.h5ad) produced by tb.preprocess_transcriptomics
- Neighbor graph built on the preprocessed gene-expression representation (e.g., PCA space)
- Pipeline configuration .yaml file for the clustering step

## Outputs

- AnnData .obs table with a Leiden cluster-assignment column populated for every cell
- Saved .h5ad file containing clustered cells and a saved configuration .yaml for the step

## How to apply

Load the AnnData .h5ad emitted by `tb.preprocess_transcriptomics` and build a neighbor graph in the preprocessed gene-expression representation (e.g., PCA space) as required by Leiden. Run the SPArrOW table operation `tb.leiden` (exposed in the `sparrow.tb` module) to compute graph-based community assignments, writing cluster identifiers into the AnnData `.obs` column so every cell carries its label. `tb.kmeans` is the listed alternative clustering tool when graph-based community detection is not desired. Each pipeline step stores a corresponding configuration .yaml file in the chosen output directory, and parameters can be tuned interactively on small crops via the Napari plugin before running at scale.

## Related tools

- **SPArrOW** (Provides the tb.* table-layer operations, including tb.leiden and tb.kmeans clustering on the cell-by-gene AnnData table) — https://github.com/saeyslab/napari-sparrow
- **Python** (Execution environment (Python 3.11 or newer) in which the sparrow.tb module and AnnData objects are used)
- **AnnData** (In-memory format for the cell-by-gene table; clustering labels are written into the .obs column)
- **uv** (Creates virtual environments and installs dependencies to run the SPArrOW pipeline)
- **Napari** (Plugin interface used to tune the parameters of SPArrOW steps, including table-layer steps, on small image crops)
- **Hydra** (Runs SPArrOW experiments from the CLI, including the clustering configuration step)

## Examples

```
from sparrow import tb; adata = tb.leiden(adata)
```

## Evaluation signals

- The AnnData .obs table contains a cluster-assignment column populated (non-null) for every cell/observation
- The clustered AnnData saves cleanly to disk as .h5ad and reloads without error
- Cluster labels are consistent with the neighbor graph built on the preprocessed representation (e.g., PCA space)
- A corresponding configuration .yaml file for the clustering step exists in the chosen output directory
- Every cell from the allocated cell-by-gene input table receives exactly one Leiden cluster identifier

## Limitations

- The package is still under very active development, so APIs may change
- Leiden clustering requires building a neighbor graph in the preprocessed representation (e.g., PCA space), so quality depends on the upstream preprocessing step
- tb.kmeans is only an alternative; the article does not report thresholds or cluster-count defaults, so parameters must be tuned (e.g., via the Napari plugin on small crops)

## Evidence
