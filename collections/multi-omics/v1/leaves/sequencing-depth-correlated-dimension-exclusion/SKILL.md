---
name: sequencing-depth-correlated-dimension-exclusion
description: Use when processing sci-L3-RNA/ATAC count matrices with Seurat/Signac
  and cell assignments must be validated against SNV ground truth — deep-sequenced
  cells otherwise separate on a depth-driven component.
when_to_use_negative:
- The data are not single-cell combinatorial-indexing counts (e.g. bulk RNA-seq where
  depth correction is handled by size factors rather than dimension removal)
- Dimensionality reduction has not yet been performed — there are no PC/LSI scores
  to correlate with depth
- The analysis goal is QC of sequencing depth itself (use the reads-per-cell cutoff
  filter instead, not dimension exclusion)
tools:
- name: Seurat
  role: Builds the RNA Seurat object, normalizes counts, runs PCA whose dimensions
    are screened against per-cell depth
- name: Signac
  role: TF-IDF normalization and LSI on the ATAC matrix; depth-correlated LSI components
    are excluded downstream
- name: UMAP
  role: Embedding computed from the retained, depth-filtered dimensions for each modality
- name: Python
  role: Correlating dimension scores with depth metrics, matching assignments to SNV
    ground truth, writing modality_agreement_rates.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:03f57112b71b2b932c2f2bdcc16d7bca15e1f6381d8c895677e53c765cd74a09
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Sequencing-depth-correlated dimension exclusion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A post-dimensionality-reduction QC step for sci-L3 single-cell analyses: identify PCA (RNA) or LSI (ATAC) dimensions whose embedding scores correlate with per-cell sequencing depth and remove them before UMAP and cell-type assignment, so that clustering reflects biology rather than library depth.

## When to use

Use when processing sci-L3-RNA/ATAC count matrices with Seurat/Signac and cell assignments must be validated against SNV ground truth — deep-sequenced cells otherwise separate on a depth-driven component.

## When NOT to use

- The data are not single-cell combinatorial-indexing counts (e.g. bulk RNA-seq where depth correction is handled by size factors rather than dimension removal)
- Dimensionality reduction has not yet been performed — there are no PC/LSI scores to correlate with depth
- The analysis goal is QC of sequencing depth itself (use the reads-per-cell cutoff filter instead, not dimension exclusion)

## Inputs

- sci-L3-RNA count matrix loaded into a Seurat object with cell barcode metadata
- sci-L3-ATAC count matrix loaded into a Signac object (TF-IDF normalized, LSI-reduced)
- Per-cell depth metrics (e.g. single_cell_summary_all.txt with reads per single cell)
- Depth filter cutoff (2000 reads per single cell; pipeline default is 10000)

## Outputs

- Retained PCA/LSI dimension set with depth-correlated dimensions flagged and removed
- UMAP embeddings computed only from retained dimensions
- Modality agreement metrics (modality_agreement_rates.csv: 92.6% RNA / 98.6% ATAC overall; 97.8% / 100% for high-depth cells)

## How to apply

After normalizing RNA counts (Seurat) and applying TF-IDF normalization plus LSI (Signac), compute the correlation between each PC/LSI dimension's score and per-cell sequencing depth (e.g. reads per single cell from single_cell_summary_all.txt). Retain cells above the study's cutoff of 2000 reads per single cell, then drop the dimension(s) with high depth correlation prior to UMAP embedding and clustering. This is essential because in the sci-L3-RNA/ATAC mixed human cell line experiment the depth-correlated component masked true cell-identity structure; excluding it raised RNA modality concordance with SNV ground truth from 92.6% (355/383) to 97.8% (233/238) and ATAC from 98.6% (378/383) to 100% (238/238) among high-depth cells. Document the excluded dimension index and its correlation value so the filtering is reproducible.

## Related tools

- **Seurat** (Builds the RNA Seurat object, normalizes counts, runs PCA whose dimensions are screened against per-cell depth)
- **Signac** (TF-IDF normalization and LSI on the ATAC matrix; depth-correlated LSI components are excluded downstream)
- **UMAP** (Embedding computed from the retained, depth-filtered dimensions for each modality)
- **Python** (Correlating dimension scores with depth metrics, matching assignments to SNV ground truth, writing modality_agreement_rates.csv)

## Examples

```
import numpy as np; from scipy.stats import pearsonr; depth_cor = {i: pearsonr(emb[:, i], depth)[0] for i in range(emb.shape[1])}; keep = [i for i, r in depth_cor.items() if abs(r) < 0.7]
```

## Evaluation signals

- The excluded dimension's score correlates strongly with reads per single cell, while retained dimensions do not drive depth-dependent separation on UMAP
- Cell-identity agreement with SNV ground truth reaches the expected rates: 92.6% RNA (355/383) and 98.6% ATAC (378/383) overall; 97.8% (233/238) and 100% (238/238) for high-depth cells
- Only cells above the 2000-reads-per-single-cell cutoff are carried into the depth-dimension screen
- modality_agreement_rates.csv is produced with the documented overall and high-depth concordance columns

## Limitations

- A limitation of the current sci-L3-RNA/ATAC co-assay is the low recovery of RNA reads per cell, so depth effects are pronounced and the RNA modality benefits most from this exclusion
- The pipeline's default reads-per-cell cutoff is 10000 while the study used 2000 — a mismatched cutoff changes which cells enter the depth correlation and concordance calculations
- Excluding a depth-correlated dimension discards any biological signal that co-varies with depth; the excluded dimension index must be reported for reproducibility

## Evidence

- [results] Depth-dimension exclusion improves modality-ground-truth concordance: [evidence span withheld — anchor status: not_found]
