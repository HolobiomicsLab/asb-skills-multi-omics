---
name: cross-modality-cell-assignment-agreement-evaluation
description: Use when a joint single-cell assay (e.g. sci-L3-RNA/ATAC) yields count
  matrices per modality plus per-cell SNV genotype calls, and you must quantify whether
  each modality alone distinguishes cell types correctly.
when_to_use_negative:
- Input is a joint RNA/ATAC dataset without an SNV-based or otherwise independent
  ground truth for cell identity — agreement rates cannot be computed.
- You need co-assay integration/linking of RNA and ATAC in a single embedding rather
  than independent per-modality validation against known identities.
- Data are bulk or pooled (no per-cell barcodes and per-cell depth metrics) — per-cell
  assignment agreement is undefined.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Seurat
  role: Construction of the RNA-modality object, normalization, PCA, and clustering
    of RNA counts
- name: Signac
  role: TF-IDF normalization of ATAC counts, LSI dimensionality reduction, and ATAC
    clustering
- name: UMAP
  role: Compute per-modality UMAP embeddings from retained PCA/LSI dimensions
- name: Python
  role: Matching modality assignments to SNV ground truth and writing modality_agreement_rates.csv
  repo: https://github.com/recombinationlab/sciL3Pipe
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:0fd2b3f6bd32563eb103b4d064e7b889ca40623d08de72df21d9e1a24acfba7b
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# Cross-modality cell-assignment agreement evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that independently clustered RNA and ATAC modalities from sci-L3-RNA/ATAC assign each single cell to the correct cell type by comparing modality-derived cell assignments against an SNV-based ground truth and reporting agreement rates overall and for high-sequencing-depth cells.

## When to use

Use when a joint single-cell assay (e.g. sci-L3-RNA/ATAC) yields count matrices per modality plus per-cell SNV genotype calls, and you must quantify whether each modality alone distinguishes cell types correctly.

## When NOT to use

- Input is a joint RNA/ATAC dataset without an SNV-based or otherwise independent ground truth for cell identity — agreement rates cannot be computed.
- You need co-assay integration/linking of RNA and ATAC in a single embedding rather than independent per-modality validation against known identities.
- Data are bulk or pooled (no per-cell barcodes and per-cell depth metrics) — per-cell assignment agreement is undefined.

## Inputs

- sci-L3-RNA count matrix per single cell
- sci-L3-ATAC count matrix per single cell
- Per-cell sequencing-depth metrics (e.g. single_cell_summary_all.txt)
- SNV-based ground-truth cell-identity calls per cell barcode

## Outputs

- modality_agreement_rates.csv (agreement rates per modality, overall and high-depth)
- Per-modality UMAP embeddings and cluster/cell-type assignments
- List of PCA/LSI dimensions excluded for correlation with sequencing depth

## How to apply

Build a Seurat object from the sci-L3-RNA counts and a Signac object from the sci-L3-ATAC counts, attaching per-cell depth metadata (retaining cells above the 2000 reads per single cell cutoff from single_cell_summary_all.txt). Normalize RNA counts and run PCA; normalize ATAC counts with TF-IDF and run LSI. Exclude any PCA/LSI dimension whose scores correlate with per-cell sequencing depth, then compute UMAP embeddings from the retained dimensions and cluster each modality separately. Match each modality's cell assignments to the SNV-based ground truth identities and compute agreement rates overall and stratified by sequencing depth. Correct application reproduces 92.6% RNA (355/383) and 98.6% ATAC (378/383) agreement overall, rising to 97.8% RNA (233/238) and 100% ATAC (238/238) for high-depth cells after depth-correlated dimension removal.

## Related tools

- **Seurat** (Construction of the RNA-modality object, normalization, PCA, and clustering of RNA counts)
- **Signac** (TF-IDF normalization of ATAC counts, LSI dimensionality reduction, and ATAC clustering)
- **UMAP** (Compute per-modality UMAP embeddings from retained PCA/LSI dimensions)
- **Python** (Matching modality assignments to SNV ground truth and writing modality_agreement_rates.csv) — https://github.com/recombinationlab/sciL3Pipe

## Examples

```
python compute_modality_agreement.py --rna rna_counts.mtx --atac atac_counts.mtx --depth single_cell_summary_all.txt --cutoff 2000 --snv_truth snv_identities.csv --out modality_agreement_rates.csv
```

## Evaluation signals

- Overall agreement against SNV ground truth reaches 92.6% (355/383) for RNA and 98.6% (378/383) for ATAC.
- High-sequencing-depth subset (cells above the 2000 reads per single cell cutoff) reaches 97.8% RNA (233/238) and 100% ATAC (238/238) agreement.
- PCA (RNA) and LSI (ATAC) dimensions whose scores correlate with per-cell sequencing depth are identified and excluded before UMAP/clustering.
- modality_agreement_rates.csv is produced with rates broken out overall and for high-depth cells.
- Cell counts used in each agreement rate are traceable to the filtered set of cells passing the per-cell read cutoff.

## Limitations

- Low recovery of RNA reads per cell in the sci-L3-RNA/ATAC co-assay reduces RNA-modality agreement relative to ATAC, motivating the high-depth stratification.
- Agreement depends on removal of the PCA/LSI dimension correlated with sequencing depth; retaining it degrades high-depth concordance.
- Adding additional rounds of cell barcoding under the sci framework reduces per-cell coverage and thus the number of recovered single cells available for evaluation.

## Evidence

- [results] Clustering each modality separately against SNV ground truth, 92.6% of cells (355/383) were correctly assigned by RNA and 98.6% by ATAC (378/383): [evidence span withheld — anchor status: not_found]
- [results] High-depth concordance after depth-dimension exclusion: [evidence span withheld — anchor status: not_found]
- [intro] ATAC readout obtained by protocol modification: "The chromatin accessibility readout was obtained by omitting the nucleosome depletion step before Tn5 barcoding and fragmentation"
- [discussion] Low RNA recovery limitation motivating depth stratification: "However, a limitation of the current approach is the low recovery of RNA reads per cell."
