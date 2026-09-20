---
name: anndata-obs-annotation-handling
description: Use when a preprocessed cell-by-gene AnnData (.h5ad) table from SPArrOW
  transcriptomics preprocessing (tb.preprocess_transcriptomics) needs graph-based
  cluster assignments written into its .obs annotation column.
when_to_use_negative:
- Input is a raw transcript allocation or binned-count table that has not yet gone
  through tb.preprocess_transcriptomics
- The analysis goal is gene scoring or neighborhood enrichment (use tb.score_genes
  or tb.nhood_enrichment) rather than assigning cells to clusters
- No neighbor graph can be built because the AnnData lacks a preprocessed expression
  representation (e.g., PCA embedding)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SPArrOW
  role: Provides the tb.leiden table operation (in the sparrow.tb module) that performs
    the clustering whose assignments are annotated into .obs
  repo: https://github.com/saeyslab/napari-sparrow
- name: AnnData
  role: Cell-by-gene table format whose .obs annotation table receives the per-cell
    cluster labels
- name: Python
  role: Runtime environment in which the SPArrOW tb.leiden call and AnnData I/O are
    executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.07.04.601829
    title: 'SPArrOW: a flexible, interactive and scalable pipeline for spatial transcriptomics
      analysis'
  source_content_hash: sha256:5c7bc01e8be0179e57970c4153e99fb7cf8436fef25d47483dbb7bfe6403c114
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# anndata-obs-annotation-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Skill for attaching clustering results (e.g., Leiden community assignments computed by SPArrOW's tb.leiden) to the .obs annotation table of a cell-by-gene AnnData object, so each cell carries its cluster label downstream.

## When to use

Use when a preprocessed cell-by-gene AnnData (.h5ad) table from SPArrOW transcriptomics preprocessing (tb.preprocess_transcriptomics) needs graph-based cluster assignments written into its .obs annotation column.

## When NOT to use

- Input is a raw transcript allocation or binned-count table that has not yet gone through tb.preprocess_transcriptomics
- The analysis goal is gene scoring or neighborhood enrichment (use tb.score_genes or tb.nhood_enrichment) rather than assigning cells to clusters
- No neighbor graph can be built because the AnnData lacks a preprocessed expression representation (e.g., PCA embedding)

## Inputs

- Preprocessed cell-by-gene AnnData table (.h5ad) produced by tb.preprocess_transcriptomics
- Neighbor graph built on the preprocessed gene-expression representation (e.g., PCA space)

## Outputs

- AnnData object whose .obs table contains a cluster-assignment column populated for every cell (observation)
- Clustered AnnData saved back to .h5ad on disk

## How to apply

Load the preprocessed cell-by-gene AnnData layer emitted by tb.preprocess_transcriptomics from its .h5ad file. Build the neighbor graph on the preprocessed gene-expression representation (e.g., PCA space), since Leiden clustering operates on a graph rather than raw counts. Run the SPArrOW table operation tb.leiden (exposed in the sparrow.tb module) on the AnnData object to compute graph-based community assignments; tb.kmeans is the listed alternative if a non-graph clustering is preferred. Write the resulting cluster identifiers into the AnnData .obs table so every observation (cell) carries its cluster label. Validate that .obs now contains a cluster-assignment column populated for every cell and that the annotated AnnData saves cleanly back to .h5ad.

## Related tools

- **SPArrOW** (Provides the tb.leiden table operation (in the sparrow.tb module) that performs the clustering whose assignments are annotated into .obs) — https://github.com/saeyslab/napari-sparrow
- **AnnData** (Cell-by-gene table format whose .obs annotation table receives the per-cell cluster labels)
- **Python** (Runtime environment in which the SPArrOW tb.leiden call and AnnData I/O are executed)

## Evaluation signals

- The AnnData .obs table contains a cluster-assignment column with a value for every cell (observation), with no missing labels
- The annotated AnnData saves cleanly to disk as .h5ad and reloads without error
- Cluster identifiers are discrete/categorical community assignments, consistent with Leiden graph-based output rather than continuous expression values
- The clustering consumed the neighbor graph built on the preprocessed (post-tb.preprocess_transcriptomics) representation, not raw counts

## Limitations

- The napari-sparrow package is under very active development, so the tb.leiden API surface may change
- tb.leiden operates on a cell-by-gene AnnData table produced after transcriptomics preprocessing; it is not designed for raw or unallocated data
- tb.kmeans is listed as an alternative clustering tool when a graph-free assignment is preferred

## Evidence

- [other] The clustering step of the SPArrOW pipeline is carried out by the table tool `tb.leiden`, which operates on the cell-by-gene AnnData table: [evidence span withheld — anchor status: not_found]
- [methods] SPArrOW is a library for single-cell Spatial Transcriptomics data analysis.: "SPArrOW is a library for single-cell Spatial Transcriptomics data analysis."
- [readme] Note: This package is still under very active development.: "Note: This package is still under very active development."
