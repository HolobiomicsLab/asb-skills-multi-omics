---
name: umap-embedding-visualization
description: Use when single-cell count matrices from sci-L3-RNA or sci-L3-ATAC need
  2D embedding for cluster visualization or when cell-type assignments must be validated
  against an independent SNV ground truth.
when_to_use_negative:
- The input is already a reduced-dimension embedding or a feature/cluster table rather
  than raw per-cell count matrices
- The data are not single-cell sci-L3 modalities (e.g. bulk Hi-C or Strand-seq libraries),
  where PCA/LSI-on-counts plus UMAP against an SNV cell-identity ground truth does
  not apply
- No per-cell depth metric is available, making the required exclusion of depth-correlated
  dimensions impossible
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Constructs the RNA Seurat object, normalizes RNA counts, runs PCA, and computes
    the UMAP embedding for the RNA modality
- name: Signac
  role: Constructs the ATAC Signac object, applies TF-IDF normalization, runs LSI,
    and supports the UMAP embedding for the chromatin-accessibility modality
- name: UMAP
  role: Nonlinear embedding method applied to retained PCA/LSI dimensions to visualize
    modality-specific cell clusters
- name: Python
  role: Used alongside Seurat/Signac to compute agreement metrics between modality
    assignments and SNV ground truth and write modality_agreement_rates.csv
  repo: https://github.com/recombinationlab/sciL3Pipe
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:3695e9ef9170c5d1b7e8c85c57ed32c713f4423a5a61e77ca8ee4b845024ce92
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# umap-embedding-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute and visualize UMAP embeddings of single-cell sci-L3 data by projecting cells from retained dimensionality-reduction space (PCA for RNA, LSI for ATAC) while excluding dimensions correlated with per-cell sequencing depth, so modality-derived clusters can be compared against SNV-based ground-truth cell identities.

## When to use

Use when single-cell count matrices from sci-L3-RNA or sci-L3-ATAC need 2D embedding for cluster visualization or when cell-type assignments must be validated against an independent SNV ground truth.

## When NOT to use

- The input is already a reduced-dimension embedding or a feature/cluster table rather than raw per-cell count matrices
- The data are not single-cell sci-L3 modalities (e.g. bulk Hi-C or Strand-seq libraries), where PCA/LSI-on-counts plus UMAP against an SNV cell-identity ground truth does not apply
- No per-cell depth metric is available, making the required exclusion of depth-correlated dimensions impossible

## Inputs

- sci-L3-RNA gene-by-cell count matrix with cell barcode metadata
- sci-L3-ATAC peak/fragment count matrix with cell barcode metadata
- Per-cell sequencing depth metrics (e.g. single_cell_summary_all.txt)
- SNV-based ground-truth cell-identity assignments per cell

## Outputs

- UMAP 2D embedding coordinates per cell for each modality
- Per-modality cell-type assignments after clustering
- modality_agreement_rates.csv with overall and high-depth concordance vs SNV ground truth

## How to apply

Build a Seurat object for the sci-L3-RNA counts and a Signac object for the sci-L3-ATAC counts, attaching per-cell barcode metadata and depth metrics (retain cells above the 2000 reads per single-cell cutoff from single_cell_summary_all.txt). Normalize RNA counts and run PCA; for ATAC, normalize with TF-IDF and run LSI in Signac. Before embedding, identify and exclude reduced dimensions whose scores correlate with per-cell sequencing depth — this is a required decision point, since a depth-driven component masks true cell-identity structure. Compute UMAP on the retained dimensions for each modality, then compare modality-derived cell assignments to the SNV ground truth to quantify agreement. Correct application is judged by recovering the paper's agreement rates (92.6% RNA, 98.6% ATAC overall; 97.8% RNA, 100% ATAC among high-depth cells) and by UMAP clusters that separate the human cell lines in the mixed experiment.

## Related tools

- **Seurat** (Constructs the RNA Seurat object, normalizes RNA counts, runs PCA, and computes the UMAP embedding for the RNA modality)
- **Signac** (Constructs the ATAC Signac object, applies TF-IDF normalization, runs LSI, and supports the UMAP embedding for the chromatin-accessibility modality)
- **UMAP** (Nonlinear embedding method applied to retained PCA/LSI dimensions to visualize modality-specific cell clusters)
- **Python** (Used alongside Seurat/Signac to compute agreement metrics between modality assignments and SNV ground truth and write modality_agreement_rates.csv) — https://github.com/recombinationlab/sciL3Pipe

## Examples

```
seurat_obj <- RunUMAP(seurat_obj, dims = 1:30)  # after NormalizeData/RunPCA for RNA or TF-IDF/RunLSI (Signac) for ATAC
```

## Evaluation signals

- Agreement with SNV ground truth reaches 92.6% (355/383) for RNA and 98.6% (378/383) for ATAC across all cells
- For high-sequencing-depth cells, concordance reaches 97.8% (233/238) by RNA and 100% (238/238) by ATAC
- Cells retained pass the 2000 reads per single-cell cutoff from single_cell_summary_all.txt before embedding
- Any reduced dimension whose scores correlate with per-cell sequencing depth is excluded prior to UMAP; UMAP clusters separate the mixed human cell lines in both modalities
- modality_agreement_rates.csv is written with the expected target values

## Limitations

- A dimension of the PCA/LSI space correlates with sequencing depth and must be excluded, or cell-identity structure is confounded by depth
- Low recovery of RNA reads per cell is a limitation of the sci-L3-RNA/ATAC co-assay, weakening the RNA modality relative to ATAC
- Adding additional rounds of cell barcoding under the sci framework reduces the number of recovered single cells due to loss of per-cell coverage

## Evidence
