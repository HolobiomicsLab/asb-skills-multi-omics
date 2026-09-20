---
name: single-cell-rna-pca-dimensionality-reduction
description: Use when processing sci-L3 or similar single-cell RNA count matrices
  and per-cell depth metrics (e.g. cells above a 2000-read cutoff) to embed cells
  for clustering and cell-identity validation.
when_to_use_negative:
- Input is already a dimension-reduced embedding or cluster assignment table — no
  PCA is needed.
- The analysis goal is raw FASTQ-to-count-matrix processing (aligning/barcoding sci-L3
  libraries with sciL3Pipe) rather than downstream RNA dimensionality reduction.
- Data are bulk RNA-seq or lack per-cell barcode metadata needed to filter low-depth
  single cells.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: R framework used to build the Seurat object from the sci-L3-RNA counts, normalize
    them, and run PCA
- name: Signac
  role: Used to build the ATAC-modality object with TF-IDF normalization and LSI dimensionality
    reduction
- name: UMAP
  role: Computes 2D embeddings from the retained PCA/LSI dimensions for each modality
- name: Python
  role: Glue language for loading count matrices and per-cell depth metrics and writing
    agreement metrics
  repo: https://github.com/recombinationlab/sciL3Pipe
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:05638b6b40b2cafbb1ae5038a3663ed26b4d169f548947d31655ee76073e04ff
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/recombinationlab/sciL3Pipe
  repo_url: https://github.com/recombinationlab/sciL3Pipe
  tools:
  - Python
  - Seurat
  - Signac
  - UMAP
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# single-cell-rna-pca-dimensionality-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reduce single-cell RNA count matrices (e.g. sci-L3-RNA/ATAC combinatorial indexing data) to low-dimensional PCA/LSI embeddings, excluding dimensions correlated with sequencing depth, to support cell-type assignment against SNV-based ground truth identities.

## When to use

Use when processing sci-L3 or similar single-cell RNA count matrices and per-cell depth metrics (e.g. cells above a 2000-read cutoff) to embed cells for clustering and cell-identity validation.

## When NOT to use

- Input is already a dimension-reduced embedding or cluster assignment table — no PCA is needed.
- The analysis goal is raw FASTQ-to-count-matrix processing (aligning/barcoding sci-L3 libraries with sciL3Pipe) rather than downstream RNA dimensionality reduction.
- Data are bulk RNA-seq or lack per-cell barcode metadata needed to filter low-depth single cells.

## Inputs

- sci-L3-RNA cell-by-gene count matrix (combinatorial-indexing barcodes)
- per-cell sequencing depth metrics (e.g. single_cell_summary_all.txt)
- SNV-based ground-truth cell identity labels for validation
- paired sci-L3-ATAC fragment/count matrix for TF-IDF + LSI (optional parallel modality)

## Outputs

- Retained PCA (RNA) / LSI (ATAC) dimension scores with depth-correlated components removed
- UMAP embeddings per modality
- Per-modality cell-type assignments and agreement rates (e.g. modality_agreement_rates.csv)

## How to apply

Construct a Seurat object from the sci-L3-RNA count matrix with cell barcode metadata, filtering cells below the per-cell read cutoff (2000 reads in the sci-L3 workflow). Normalize the RNA counts and run PCA; for the paired ATAC modality, normalize with TF-IDF in Signac and run LSI instead. Critically, exclude any PCA/LSI dimension whose scores correlate with per-cell sequencing depth before downstream analysis — in the sci-L3-RNA/ATAC human cell line experiment this exclusion was required to avoid depth-driven artifacts. Compute UMAP embeddings from the retained dimensions, then cluster and match modality-derived cell assignments against SNV-based ground truth. Applying this correctly on the sci-L3-RNA data yielded 92.6% correct assignment (355/383) overall and 97.8% (233/238) among high-sequencing-depth cells.

## Related tools

- **Seurat** (R framework used to build the Seurat object from the sci-L3-RNA counts, normalize them, and run PCA)
- **Signac** (Used to build the ATAC-modality object with TF-IDF normalization and LSI dimensionality reduction)
- **UMAP** (Computes 2D embeddings from the retained PCA/LSI dimensions for each modality)
- **Python** (Glue language for loading count matrices and per-cell depth metrics and writing agreement metrics) — https://github.com/recombinationlab/sciL3Pipe

## Examples

```
rna <- NormalizeData(CreateSeuratObject(counts = sci_l3_rna_counts)); rna <- RunPCA(rna, verbose = FALSE)
```

## Evaluation signals

- Overall RNA cell-identity agreement vs SNV ground truth reproduces 92.6% (355/383) and ATAC 98.6% (378/383).
- High-sequencing-depth cells (above the 2000-read cutoff) reach 97.8% (RNA, 233/238) and 100% (ATAC, 238/238) agreement.
- No retained PCA/LSI dimension shows appreciable correlation with per-cell sequencing depth.
- Cells below the cutoff of 2000 reads per single cell are excluded before embedding.
- Output agreement metrics are written to modality_agreement_rates.csv with the expected columns/rates.

## Limitations

- A limitation of the sci-L3-RNA/ATAC co-assay is the low recovery of RNA reads per cell, which caps per-cell RNA signal and reduces assignment accuracy for low-depth cells.
- Omitting the depth-correlated PCA/LSI dimension is required; retaining it would confound cell-state structure with sequencing depth.
- Additional rounds of cell barcoding in the sci framework reduce the number of recovered single cells due to loss of per-cell coverage.
- The sciL3Pipe repository states no software version numbers, so exact tool versions for reproducibility are not pinned in the article context.

## Evidence
