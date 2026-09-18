---
name: seurat-object-construction-from-count-matrices
description: Use when you have sci-L3-RNA and sci-L3-ATAC count matrices plus per-cell
  depth metrics (e.g. single_cell_summary_all.txt) and need cell objects for PCA/LSI,
  UMAP, and cell-type assignment against SNV ground truth.
when_to_use_negative:
- Input is already a processed Seurat/Signac object or a dimensionally-reduced embedding
  — re-construction from count matrices is unnecessary and would redo normalization.
- You need to process raw FASTQs into count matrices — that upstream step belongs
  to the sciL3Pipe Snakemake pipeline, not to this object-construction skill.
- Your data lacks per-cell depth metrics, since the 2000-reads-per-cell cutoff and
  depth-correlated dimension exclusion are central to reproducing the reported agreement
  rates.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Constructs the RNA modality object, performs normalization and PCA
- name: Signac
  role: Constructs the ATAC modality object, applies TF-IDF normalization and LSI
- name: UMAP
  role: Computes 2D embeddings from retained PCA/LSI dimensions
- name: Python
  role: Loads sci-L3 count matrices and depth metrics (single_cell_summary_all.txt)
    into the workflow
  repo: https://github.com/recombinationlab/sciL3Pipe
merged_aliases:
- signac-object-construction-from-count-matrices
merged_alias_records:
- alias: signac-object-construction-from-count-matrices
  slug: signac-object-construction-from-count-matrices
  jaccard_score: 0.6667
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:d97b3c4c340f034ec11963b41a6018cc13a5305db8517e277d379e4bcf0caf65
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# seurat-object-construction-from-count-matrices

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds per-modality single-cell analysis objects (Seurat for RNA, Signac for ATAC) from sci-L3-RNA/ATAC count matrices with per-cell barcode metadata and sequencing-depth metrics, applying the paper's 2000-reads-per-cell cutoff before dimensional reduction.

## When to use

Use when you have sci-L3-RNA and sci-L3-ATAC count matrices plus per-cell depth metrics (e.g. single_cell_summary_all.txt) and need cell objects for PCA/LSI, UMAP, and cell-type assignment against SNV ground truth.

## When NOT to use

- Input is already a processed Seurat/Signac object or a dimensionally-reduced embedding — re-construction from count matrices is unnecessary and would redo normalization.
- You need to process raw FASTQs into count matrices — that upstream step belongs to the sciL3Pipe Snakemake pipeline, not to this object-construction skill.
- Your data lacks per-cell depth metrics, since the 2000-reads-per-cell cutoff and depth-correlated dimension exclusion are central to reproducing the reported agreement rates.

## Inputs

- sci-L3-RNA count matrix (gene x cell)
- sci-L3-ATAC count matrix (peak x cell)
- per-cell depth metrics from single_cell_summary_all.txt
- cell barcode metadata (combinatorial indexing barcodes)
- SNV-based ground-truth cell identity labels

## Outputs

- Seurat object for the RNA modality
- Signac object for the ATAC modality
- UMAP embeddings per modality
- modality_agreement_rates.csv with overall and high-depth agreement metrics

## How to apply

Load the sci-L3-RNA and sci-L3-ATAC count matrices together with per-cell depth metrics (from single_cell_summary_all.txt, retaining cells above the cutoff of 2000 reads per single cell), and construct a Seurat object for the RNA modality and a Signac object for the ATAC modality with cell barcode metadata. For RNA, normalize counts and run PCA; for ATAC, normalize with TF-IDF and run LSI in Signac. Before downstream analysis, exclude any dimension whose scores correlate with per-cell sequencing depth — a decision point that was required to reach the paper's agreement rates. Compute UMAP embeddings from the retained dimensions, then match modality-derived cell assignments against SNV-based ground truth to compute agreement rates (92.6% RNA, 98.6% ATAC overall; 97.8%/100% for high-depth cells) written to modality_agreement_rates.csv.

## Related tools

- **Seurat** (Constructs the RNA modality object, performs normalization and PCA)
- **Signac** (Constructs the ATAC modality object, applies TF-IDF normalization and LSI)
- **UMAP** (Computes 2D embeddings from retained PCA/LSI dimensions)
- **Python** (Loads sci-L3 count matrices and depth metrics (single_cell_summary_all.txt) into the workflow) — https://github.com/recombinationlab/sciL3Pipe

## Examples

```
seurat_obj <- CreateSeuratObject(counts = rna_counts, meta.data = cell_meta); seurat_obj <- subset(seurat_obj, subset = nCount_RNA > 2000); seurat_obj <- RunPCA(seurat_obj)
```

## Evaluation signals

- Cell counts after filtering: 383 cells scored against SNV ground truth (355 correct by RNA, 378 by ATAC).
- Agreement rates match published targets: 92.6% RNA / 98.6% ATAC overall; 97.8% / 100% among high-depth cells (233/238 and 238/238).
- All retained cells exceed the 2000 reads per single cell cutoff specified in the pipeline config.
- No retained PCA/LSI dimension shows a correlation with per-cell sequencing depth before UMAP and assignment.
- modality_agreement_rates.csv is produced with both overall and high-depth-cell agreement columns.

## Limitations

- The sci-L3-RNA/ATAC co-assay has low recovery of RNA reads per cell, which limits RNA modality depth and motivates the high-depth-cell subset analysis.
- Agreement metrics depend on excluding dimensions correlated with sequencing depth; skipping this step changes the reported rates.
- The sci-L3 framework loses recovered single cells with additional barcoding rounds, shrinking the evaluable cell set.

## Evidence

- [methods] Retain cells above cutoff and build modality objects: [evidence span withheld — anchor status: not_found]
- [results] Depth-correlated dimension exclusion is required before downstream analysis: [evidence span withheld — anchor status: not_found]
- [results] Published agreement rates validate correct application: [evidence span withheld — anchor status: not_found]
