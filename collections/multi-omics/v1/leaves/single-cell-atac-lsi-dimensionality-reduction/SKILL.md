---
name: single-cell-atac-lsi-dimensionality-reduction
description: Use when you have a single-cell chromatin accessibility (scATAC) count
  matrix from a combinatorial-indexing assay (e.
when_to_use_negative:
- Input is single-cell RNA-seq data where plain PCA on normalized counts suffices
  — use the RNA-modality Seurat/PCA path instead
- The matrix is bulk or pseudobulk accessibility data without per-cell barcodes, so
  no per-cell depth metrics exist to gate the 2000 reads/cell cutoff
- Downstream analysis requires all latent dimensions regardless of confounding, e.g.
  when explicitly studying technical artifacts rather than cell identity
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: Signac
  role: TF-IDF normalization of ATAC counts and LSI dimensionality reduction on the
    accessibility matrix
- name: UMAP
  role: Nonlinear embedding of retained LSI dimensions for visualization and clustering
- name: Seurat
  role: Counterpart framework used to build the RNA-modality object and run PCA for
    cross-modality comparison
- name: Python
  role: Glue language for loading count matrices, depth metrics, and writing modality_agreement_rates.csv
- name: sciL3Pipe
  role: Snakemake pipeline producing the aligned, demultiplexed sci-L3 count data
    that feed the LSI analysis
  repo: github.com/recombinationlab/sciL3Pipe
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:21fe9e3381bbe0287831b070ff9c63333562241e86ee6aa7af08d4bddc59ec46
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/recombinationlab/sciL3Pipe
  tools:
  - Python
  - Seurat
  - Signac
  - UMAP
  - sciL3Pipe
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# single-cell-atac-lsi-dimensionality-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Latent semantic indexing (LSI) dimensionality reduction on single-cell ATAC count matrices as practiced in the sci-L3-RNA/ATAC analysis: TF-IDF-normalized accessibility counts are projected via LSI in Signac, depth-correlated dimensions are excluded, and the retained dimensions are used for UMAP and cell-type assignment against SNV-based ground truth.

## When to use

Use when you have a single-cell chromatin accessibility (scATAC) count matrix from a combinatorial-indexing assay (e.g. sci-L3-RNA/ATAC) and need to reduce it to interpretable latent dimensions for clustering/UMAP, especially when per-cell sequencing-depth confounding must be controlled before comparing cell identities across modalities.

## When NOT to use

- Input is single-cell RNA-seq data where plain PCA on normalized counts suffices — use the RNA-modality Seurat/PCA path instead
- The matrix is bulk or pseudobulk accessibility data without per-cell barcodes, so no per-cell depth metrics exist to gate the 2000 reads/cell cutoff
- Downstream analysis requires all latent dimensions regardless of confounding, e.g. when explicitly studying technical artifacts rather than cell identity

## Inputs

- sci-L3-ATAC single-cell count matrix (cell barcode x genomic bins/peaks)
- Per-cell sequencing depth metrics (e.g. from single_cell_summary_all.txt)
- Cell barcode metadata
- Optional: SNV-based ground truth cell identity labels for validation

## Outputs

- LSI reduced-dimension representation of the ATAC modality with depth-correlated dimensions excluded
- UMAP embedding computed from retained LSI dimensions
- Per-cell cluster/identity assignments
- Agreement metrics versus ground truth (e.g. modality_agreement_rates.csv with 98.6% overall, 100% high-depth)

## How to apply

Build a Signac object from the sci-L3-ATAC count matrix with per-cell barcode metadata and depth metrics (retaining cells above the 2000 reads per single cell cutoff from the pipeline config). Normalize the ATAC counts with TF-IDF and run LSI. Critically, inspect each LSI dimension's correlation with per-cell sequencing depth and exclude correlated dimensions before downstream analysis, since in this dataset one dimension tracked depth rather than biology. Compute UMAP embeddings from the retained dimensions, then cluster and match modality-derived cell assignments against an independent ground truth (here, SNV-based cell identities) to compute agreement rates. In the reference implementation this yielded 98.6% ATAC agreement overall (378/383) and 100% (238/238) for high-depth cells, versus 92.6%/97.8% for the RNA modality processed analogously in Seurat.

## Related tools

- **Signac** (TF-IDF normalization of ATAC counts and LSI dimensionality reduction on the accessibility matrix)
- **UMAP** (Nonlinear embedding of retained LSI dimensions for visualization and clustering)
- **Seurat** (Counterpart framework used to build the RNA-modality object and run PCA for cross-modality comparison)
- **Python** (Glue language for loading count matrices, depth metrics, and writing modality_agreement_rates.csv)
- **sciL3Pipe** (Snakemake pipeline producing the aligned, demultiplexed sci-L3 count data that feed the LSI analysis) — github.com/recombinationlab/sciL3Pipe

## Examples

```
atac <- RunTFIDF(atac_signac_obj); atac <- RunLSI(atac, n = 30); atac <- RunUMAP(atac, reduction = "lsi", dims = 2:30)
```

## Evaluation signals

- Cells retained pass the reads-per-single-cell cutoff of 2000 from the pipeline config before reduction
- No retained LSI dimension's scores correlate with per-cell sequencing depth (the depth-correlated dimension is explicitly excluded)
- ATAC-modality clustering agrees with SNV-based ground truth at ~98.6% overall (378/383) and ~100% for high-sequencing-depth cells (238/238)
- Agreement metrics are written to modality_agreement_rates.csv in the expected overall/high-depth structure
- UMAP embedding visually separates the mixed human cell line types consistently with the SNV-defined identities

## Limitations

- At least one LSI dimension is correlated with per-cell sequencing depth and must be excluded; keeping it confounds downstream clustering
- The companion sci-L3-RNA/ATAC co-assay recovers few RNA reads per cell, so cross-modality comparisons are bounded by RNA data quality
- Pipeline configs ship without stated software version numbers, and the repository has no changelog, complicating exact reproducibility of the reduction environment

## Evidence

- [results] ATAC agreement finding: [evidence span withheld — anchor status: not_found]
- [results] Depth-correlated dimension exclusion: [evidence span withheld — anchor status: not_found]
