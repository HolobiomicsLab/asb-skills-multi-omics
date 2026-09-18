---
name: transcriptomics-count-matrix-preprocessing-consumption
description: Use when a preprocessed cell-by-gene AnnData table (.h5ad, output of
  tb.preprocess_transcriptomics) must be consumed for downstream analysis, specifically
  graph-based clustering of cells into communities.
when_to_use_negative:
- The input is raw transcript counts that have not yet passed tb.preprocess_transcriptomics
  (or the preceding tb.allocate / tb.bin_counts allocation steps) — preprocessing
  must come first.
- The analysis target is the image or segmentation stage of the SPArrOW pipeline (e.g.
  im.segment, im.tiling_correction) rather than the cell-by-gene expression table.
- The goal is gene-level scoring (tb.score_genes) or spatial neighborhood enrichment
  (tb.nhood_enrichment) rather than per-cell cluster assignment.
- The input is a proteomics table processed with tb.preprocess_proteomics rather than
  transcriptomics data.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SPArrOW
  role: Provides the table tool tb.leiden (and alternative tb.kmeans) that performs
    clustering on the preprocessed cell-by-gene AnnData table
  repo: https://github.com/saeyslab/napari-sparrow
- name: Python
  role: Runtime for running the sparrow.tb module (requires Python 3.11 or newer);
    the preprocessed table and cluster assignments are manipulated as AnnData objects
    in Python
  repo: https://github.com/saeyslab/napari-sparrow
- name: AnnData
  role: Container format for the cell-by-gene table; cluster identifiers are written
    into the .obs column of this object
  repo: https://github.com/saeyslab/napari-sparrow
- name: uv
  role: Used to create virtual environments and install dependencies when running
    the SPArrOW pipeline in Python
  repo: https://github.com/saeyslab/napari-sparrow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.07.04.601829
    title: 'SPArrOW: a flexible, interactive and scalable pipeline for spatial transcriptomics
      analysis'
  source_content_hash: sha256:970b0ae6675f65d1e78f097878c086e4e21c201a20b332a825ef0811a7aaad6b
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
  - Python
  - SPArrOW
  - uv
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 9
---

# transcriptomics-count-matrix-preprocessing-consumption

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Consumes the preprocessed cell-by-gene AnnData table produced by the SPArrOW pipeline step tb.preprocess_transcriptomics and derives downstream structure from it, chiefly graph-based community clustering via the table tool tb.leiden (with tb.kmeans as an alternative), storing cluster assignments in the AnnData .obs table.

## When to use

Use when a preprocessed cell-by-gene AnnData table (.h5ad, output of tb.preprocess_transcriptomics) must be consumed for downstream analysis, specifically graph-based clustering of cells into communities.

## When NOT to use

- The input is raw transcript counts that have not yet passed tb.preprocess_transcriptomics (or the preceding tb.allocate / tb.bin_counts allocation steps) — preprocessing must come first.
- The analysis target is the image or segmentation stage of the SPArrOW pipeline (e.g. im.segment, im.tiling_correction) rather than the cell-by-gene expression table.
- The goal is gene-level scoring (tb.score_genes) or spatial neighborhood enrichment (tb.nhood_enrichment) rather than per-cell cluster assignment.
- The input is a proteomics table processed with tb.preprocess_proteomics rather than transcriptomics data.

## Inputs

- Preprocessed cell-by-gene AnnData table (.h5ad file), output of tb.preprocess_transcriptomics
- Neighbor graph built on the preprocessed gene-expression representation (e.g. PCA space)

## Outputs

- AnnData object with a cluster-assignment column in the .obs table (Leiden community labels per cell)
- Clustered AnnData saved to disk (.h5ad)

## How to apply

Load the preprocessed cell-by-gene AnnData layer emitted by the transcriptomics preprocessing step (tb.preprocess_transcriptomics) from its .h5ad file. Build the neighbor graph on the preprocessed gene-expression representation (e.g. PCA space), since Leiden clustering is graph-based and requires this graph as input. Run the SPArrOW table operation tb.leiden, exposed in the sparrow.tb module, on the AnnData object to compute graph-based community assignments; tb.kmeans is the listed alternative if a non-graph clustering is preferred. Write the resulting Leiden cluster identifiers into the AnnData .obs column so each cell (observation) carries its cluster label. Validate that the .obs table now contains a cluster-assignment column populated for every cell and that the clustered AnnData saves cleanly to disk.

## Related tools

- **SPArrOW** (Provides the table tool tb.leiden (and alternative tb.kmeans) that performs clustering on the preprocessed cell-by-gene AnnData table) — https://github.com/saeyslab/napari-sparrow
- **Python** (Runtime for running the sparrow.tb module (requires Python 3.11 or newer); the preprocessed table and cluster assignments are manipulated as AnnData objects in Python) — https://github.com/saeyslab/napari-sparrow
- **AnnData** (Container format for the cell-by-gene table; cluster identifiers are written into the .obs column of this object) — https://github.com/saeyslab/napari-sparrow
- **uv** (Used to create virtual environments and install dependencies when running the SPArrOW pipeline in Python) — https://github.com/saeyslab/napari-sparrow

## Examples

```
from sparrow import tb; tb.leiden(adata)  # adata = preprocessed cell-by-gene AnnData loaded from .h5ad; labels land in adata.obs
```

## Evaluation signals

- The AnnData .obs table contains a cluster-assignment (Leiden) column populated for every cell (observation) — no null labels.
- The clustered AnnData object saves cleanly to disk as .h5ad and reloads without loss of the cluster column.
- The number of Leiden communities is finite and consistent with a graph built on the preprocessed expression representation (e.g. PCA space).
- Cross-checking against the alternative tool tb.kmeans yields a comparable partition structure on the same preprocessed table.
- The consuming step is downstream of tb.preprocess_transcriptomics: the input .h5ad already reflects the preprocessed (not raw) transcript counts.

## Limitations

- The SPArrOW package is 'still under very active development', so the tb.leiden API and behavior may change between versions.
- Leiden clustering cannot run directly on the raw table: a neighbor graph on the preprocessed representation (e.g. PCA space) must be built first.
- SPArrOW requires Python 3.11 or newer, and the project uses uv to manage virtual environments and dependencies.
- Leiden is graph-based; tb.kmeans is listed as the alternative when a graph-free clustering is desired.

## Evidence
