---
name: leiden-graph-clustering-of-cell-by-gene-matrix
description: Use when a preprocessed cell-by-gene AnnData table (output of tb.preprocess_transcriptomics)
  needs unsupervised cluster assignments per cell in a SPArrOW spatial transcriptomics
  workflow.
when_to_use_negative:
- The table has not yet been through transcriptomics preprocessing (tb.preprocess_transcriptomics)
  — cluster on the preprocessed representation instead of raw counts.
- The goal is gene scoring or neighborhood enrichment (use tb.score_genes or tb.nhood_enrichment)
  rather than per-cell cluster labels.
- Input is a protein table — the SPArrOW pipeline exposes tb.preprocess_proteomics
  separately, and this skill targets the cell-by-gene transcriptomics layer.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
tools:
- name: SPArrOW
  role: Provides the table tool tb.leiden that performs Leiden clustering on the cell-by-gene
    AnnData table
  repo: https://github.com/saeyslab/napari-sparrow
- name: Python
  role: Runtime environment (Python 3.11 or newer) in which sparrow.tb.leiden is invoked
  repo: https://github.com/saeyslab/napari-sparrow
- name: AnnData
  role: Container format for the cell-by-gene table; cluster labels are written into
    the .obs table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.07.04.601829
    title: 'SPArrOW: a flexible, interactive and scalable pipeline for spatial transcriptomics
      analysis'
  source_content_hash: sha256:fb6116e48f1a1bad6fdc739402d626064239ba7f5f8e6eebbe7c2e749f6b3e1b
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# leiden-graph-clustering-of-cell-by-gene-matrix

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Performs graph-based community detection on a preprocessed cell-by-gene expression table using the SPArrOW table tool `tb.leiden`, assigning each cell a Leiden cluster label in the AnnData .obs table. It is the clustering step of the SPArrOW spatial transcriptomics pipeline, with `tb.kmeans` as an alternative.

## When to use

Use when a preprocessed cell-by-gene AnnData table (output of tb.preprocess_transcriptomics) needs unsupervised cluster assignments per cell in a SPArrOW spatial transcriptomics workflow.

## When NOT to use

- The table has not yet been through transcriptomics preprocessing (tb.preprocess_transcriptomics) — cluster on the preprocessed representation instead of raw counts.
- The goal is gene scoring or neighborhood enrichment (use tb.score_genes or tb.nhood_enrichment) rather than per-cell cluster labels.
- Input is a protein table — the SPArrOW pipeline exposes tb.preprocess_proteomics separately, and this skill targets the cell-by-gene transcriptomics layer.

## Inputs

- Preprocessed cell-by-gene AnnData table (.h5ad) produced by tb.preprocess_transcriptomics

## Outputs

- AnnData object with a Leiden cluster-assignment column in .obs, populated for every cell
- Clustered AnnData saved to disk as .h5ad

## How to apply

Load the preprocessed cell-by-gene AnnData table (emitted by `tb.preprocess_transcriptomics`) from its .h5ad file. Build the neighbor graph on the preprocessed gene-expression representation (e.g. PCA space), as required by Leiden clustering. Run `tb.leiden` from the `sparrow.tb` module on the AnnData object to compute graph-based community assignments, writing Leiden cluster identifiers into the AnnData .obs column so every cell (observation) carries its cluster label. If a simpler partition is preferred, `tb.kmeans` is the SPArrOW alternative listed for the clustering step.

## Related tools

- **SPArrOW** (Provides the table tool tb.leiden that performs Leiden clustering on the cell-by-gene AnnData table) — https://github.com/saeyslab/napari-sparrow
- **Python** (Runtime environment (Python 3.11 or newer) in which sparrow.tb.leiden is invoked) — https://github.com/saeyslab/napari-sparrow
- **AnnData** (Container format for the cell-by-gene table; cluster labels are written into the .obs table)

## Examples

```
uv run python -c "import sparrow; adata = sparrow.tb.leiden(adata)"
```

## Evaluation signals

- The .obs table of the AnnData object contains a cluster-assignment column populated for every cell (observation).
- The clustered AnnData saves cleanly to disk as .h5ad without errors.
- Each cell carries exactly one Leiden cluster label (no NaN or missing assignments).
- Cluster assignments are consistent with the neighbor graph built on the preprocessed gene-expression (e.g. PCA) representation.

## Limitations

- The napari-sparrow package is still under very active development, so API details may change.
- tb.leiden requires a neighbor graph built on the preprocessed representation; running it without this graph step is unsupported.
- tb.kmeans is the listed alternative; graph-based Leiden may not be the right choice for all datasets.

## Evidence
