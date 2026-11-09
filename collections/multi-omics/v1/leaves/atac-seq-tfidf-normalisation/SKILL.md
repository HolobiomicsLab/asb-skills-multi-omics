---
name: atac-seq-tfidf-normalisation
description: Use when raw single-cell ATAC-seq peak-count matrices require normalisation
  before dimensionality reduction (e.g. PCA on LSI components), especially within
  multimodal (RNA+ATAC) MuData workflows.
when_to_use_negative:
- The data is RNA-seq counts — use scanpy's normalise_total/log1p preprocessing instead,
  as in the paper's per-modality RNA pipeline
- The ATAC counts are already transformed (e.g. binarised and LSI-computed elsewhere)
  and re-applying TF-IDF would double-transform the matrix
- The assay is not chromatin accessibility data — TF-IDF here is specific to MUON's
  ATAC module and its peak-count representation
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3168
tools:
- name: MUON
  role: Provides the ATAC module function ac.pp.tfidf that performs the TF-IDF transformation
    with a configurable scale_factor on the ATAC AnnData
  repo: https://github.com/scverse/muon
- name: SCANPY
  role: Used downstream to compute principal components and UMAP embeddings from the
    TF-IDF-normalised ATAC data
- name: NumPy
  role: Array computation backend used when exporting and validating the embedding
    coordinates
- name: pandas
  role: Used to export the per-cell embedding coordinates (including the ATAC-PC UMAP)
    to a CSV table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:3fcdfdc0cac99e2387bb21075029a47ca3feb442b301b478b13c483fd8d53803
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/muon
  tools:
  - MUON
  - NumPy
  - SCANPY
  - pandas
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# atac-seq-tfidf-normalisation

## Summary

Normalise single-cell ATAC-seq count data using the TF-IDF (term frequency–inverse document frequency) transformation implemented in MUON's ATAC module, treating peaks as terms and cells as documents, typically followed by PCA for downstream embedding.

## When to use

Use when raw single-cell ATAC-seq peak-count matrices require normalisation before dimensionality reduction (e.g. PCA on LSI components), especially within multimodal (RNA+ATAC) MuData workflows.

## When NOT to use

- The data is RNA-seq counts — use scanpy's normalise_total/log1p preprocessing instead, as in the paper's per-modality RNA pipeline
- The ATAC counts are already transformed (e.g. binarised and LSI-computed elsewhere) and re-applying TF-IDF would double-transform the matrix
- The assay is not chromatin accessibility data — TF-IDF here is specific to MUON's ATAC module and its peak-count representation

## Inputs

- ATAC modality AnnData object with a cells x peaks count matrix (e.g. inside a MuData loaded via mu.read from 10x multiome data)
- Optional: nucleosome signal and TSS enrichment QC metrics (ac.tl.nucleosome_signal, ac.tl.tss_enrichment)

## Outputs

- TF-IDF-normalised ATAC expression matrix stored in the ATAC AnnData (in place)
- ATAC PCA/LSI components and 2D UMAP embedding derived from the normalised matrix

## How to apply

Load the ATAC modality (e.g. from a 10x multiome dataset read with mu.read into a MuData object), then apply TF-IDF in place on the ATAC AnnData using muon's ATAC module with the scale factor set to 1e4 (ac.pp.tfidf(atac, scale_factor=1e4)). TF-IDF is appropriate for sparse count data with large differences in peak counts per cell, as it down-weights broadly accessible peaks. After transformation, compute principal components per modality with scanpy and run scanpy UMAP to obtain 2D embeddings from ATAC (LSI) PCs; these can subsequently be combined with the RNA modality via WNN neighbours (mu.pp.neighbors) or MOFA factors. Optional upstream ATAC quality-control steps include binarisation (ac.pp.binarize), nucleosome signal estimation, and TSS enrichment, but TF-IDF is the core normalisation step before PC computation.

## Related tools

- **MUON** (Provides the ATAC module function ac.pp.tfidf that performs the TF-IDF transformation with a configurable scale_factor on the ATAC AnnData) — https://github.com/scverse/muon
- **SCANPY** (Used downstream to compute principal components and UMAP embeddings from the TF-IDF-normalised ATAC data)
- **NumPy** (Array computation backend used when exporting and validating the embedding coordinates)
- **pandas** (Used to export the per-cell embedding coordinates (including the ATAC-PC UMAP) to a CSV table)

## Examples

```
import muon.atac as ac; ac.pp.tfidf(atac, scale_factor=1e4)
```

## Evaluation signals

- After transformation, 'X_mofa' or the expected PC/LSI keys are present in mdata.obsm when the downstream joint embedding is built
- The exported CSV contains one row per cell with exactly two finite numeric UMAP coordinate columns per embedding, including the ATAC-PC UMAP
- UMAP plots of ATAC (LSI) PCs, coloured by coarse-grained cell type, show separation of cell populations comparable to the RNA-PC UMAP
- The call completes in place on the ATAC modality (mdata['atac']) without altering the RNA modality of the MuData object

## Limitations

- TF-IDF in MUON's ATAC module is designed for ATAC peak/count data; other modalities in the MuData object need their own normalisation (e.g. scanpy normalisation for RNA, dsb or CLR for protein)
- The quality of the resulting ATAC embedding depends on upstream QC (nucleosome signal, TSS enrichment, binarisation choices) which is not handled by TF-IDF itself

## Evidence

- [methods] TF-IDF transformation is implemented in the muon's ATAC module: "TF-IDF transformation is implemented in the muon's ATAC module"
- [methods] scale factor 1e4: "ac.pp.tfidf(atac, scale_factor=1e4)"
- [readme] muon description: "`muon` is a multimodal omics Python framework."
- [abstract] per-modality analysis rationale: "MUON enables a versatile range of analyses, from data preprocessing to flexible multi-omics alignment."
