---
name: single-cell-pca-dimensionality-reduction
description: Use when a MuData or AnnData single-cell object needs per-modality low-dimensional
  representations (e.g. RNA PCs, ATAC LSI PCs) before UMAP plotting or before building
  joint WNN/MOFA embeddings.
when_to_use_negative:
- Input is already a low-dimensional embedding or factor table (e.g. X_mofa is present
  in mdata.obsm) — skip directly to neighbours/UMAP
- Single unimodal ATAC data where a dedicated latent-space method such as SnapATAC's
  dimensionality reduction is more appropriate
- Modalities whose counts lack appropriate per-modality preprocessing (RNA not normalised,
  ATAC not TF-IDF transformed) — PCA on raw counts is not the intended workflow
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MUON
  role: Framework that organises the multimodal data (MuData), provides TF-IDF preprocessing
    for ATAC (ac.pp.tfidf), and hosts the joint integration steps built on the per-modality
    PCs
  repo: https://github.com/scverse/muon
- name: SCANPY
  role: Performs normalisation of the RNA modality, PCA computation per modality,
    and UMAP embedding from the PC space
- name: MOFA+
  role: Fits multi-omic factor analysis (mu.tl.mofa, use_var='highly_variable') to
    produce joint factors (X_mofa) as an alternative joint embedding source
- name: pandas
  role: Exports the per-cell UMAP coordinates from the embeddings to a CSV table for
    validation
- name: NumPy
  role: Supports numeric validation of embedding coordinate columns (finite numeric
    values, two columns per embedding)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:47d158fe7de652c7f30f8c28a946ae52ac88398b218dc8212782c03ca45bd8a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/muon
  tools:
  - MOFA+
  - MUON
  - NumPy
  - SCANPY
  - pandas
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2
n_claims: 10
---

# single-cell-pca-dimensionality-reduction

## Summary

Performs per-modality PCA-based dimensionality reduction on single-cell multi-omics data (RNA counts and TF-IDF-transformed scATAC peaks) to obtain low-dimensional cell embeddings that feed downstream UMAP visualisation and joint multi-omics integration in MUON workflows.

## When to use

Use when a MuData or AnnData single-cell object needs per-modality low-dimensional representations (e.g. RNA PCs, ATAC LSI PCs) before UMAP plotting or before building joint WNN/MOFA embeddings.

## When NOT to use

- Input is already a low-dimensional embedding or factor table (e.g. X_mofa is present in mdata.obsm) — skip directly to neighbours/UMAP
- Single unimodal ATAC data where a dedicated latent-space method such as SnapATAC's dimensionality reduction is more appropriate
- Modalities whose counts lack appropriate per-modality preprocessing (RNA not normalised, ATAC not TF-IDF transformed) — PCA on raw counts is not the intended workflow

## Inputs

- MuData object holding paired PBMC multiome scRNA-seq and scATAC-seq modalities (e.g. loaded with mu.read or mu.read_10x_h5)
- Raw RNA count matrix (AnnData) suitable for scanpy normalisation
- Raw scATAC peak/count matrix (AnnData) for TF-IDF transformation with ac.pp.tfidf(scale_factor=1e4)

## Outputs

- Per-modality PCA embeddings stored in modality .obsm (e.g. 'X_pca' for RNA, LSI-style PCs for ATAC)
- 2D UMAP embeddings derived from RNA PCs and ATAC PCs, stored as .obsm entries
- Joint MOFA factors (X_mofa in mdata.obsm) and WNN-based joint UMAP coordinates
- Per-cell CSV table of embedding coordinates (RNA-PC UMAP, ATAC-PC UMAP, MOFA+WNN UMAP)

## How to apply

Load the multimodal PBMC multiome dataset as a MuData object with mu.read, then preprocess each modality independently: normalise the RNA modality with scanpy and transform the ATAC modality with TF-IDF using ac.pp.tfidf(atac, scale_factor=1e4). Compute principal components per modality with scanpy (sc.pp.pca), yielding embeddings such as 'X_pca' in each modality's obsm. From these PCs, run scanpy UMAP to obtain 2D embeddings from RNA PCs and from ATAC (LSI) PCs. The rationale is MUON's modular design: existing single-omics methods (SCANPY) are reused to prepare each modality before multi-omics integration (MOFA factors via mu.tl.mofa, WNN neighbours via mu.pp.neighbors) — the per-modality PCs are also the input to the joint embeddings of Fig. 3b.

## Related tools

- **MUON** (Framework that organises the multimodal data (MuData), provides TF-IDF preprocessing for ATAC (ac.pp.tfidf), and hosts the joint integration steps built on the per-modality PCs) — https://github.com/scverse/muon
- **SCANPY** (Performs normalisation of the RNA modality, PCA computation per modality, and UMAP embedding from the PC space)
- **MOFA+** (Fits multi-omic factor analysis (mu.tl.mofa, use_var='highly_variable') to produce joint factors (X_mofa) as an alternative joint embedding source)
- **pandas** (Exports the per-cell UMAP coordinates from the embeddings to a CSV table for validation)
- **NumPy** (Supports numeric validation of embedding coordinate columns (finite numeric values, two columns per embedding))

## Examples

```
ac.pp.tfidf(atac, scale_factor=1e4); sc.pp.pca(atac); sc.pp.neighbors(rna); sc.tl.umap(rna)
```

## Evaluation signals

- Output CSV table exists and contains exactly one row per cell
- Each embedding contributes exactly two finite numeric UMAP coordinate columns per cell
- 'X_mofa' is present in mdata.obsm after running mu.tl.mofa with default use_var='highly_variable'
- UMAP plots coloured by coarse-grained cell type annotation (mdata.obs) show coherent clustering consistent with Fig. 3b
- Per-modality embeddings (RNA PCs, ATAC LSI PCs) reproduce both the individual-modality and joint (MOFA/WNN) UMAP views

## Limitations

- MUON's joint embedding support focuses on integrating modalities; handling of novel assays, genomic-coordinate based assays, and spatially resolved assays is listed as future development
- Explicitly modelling dependencies between feature sets across omics in MuData is not yet supported (planned future development)
- Quality of the ATAC PC space depends on prior TF-IDF transformation with scale_factor=1e4; skipping per-modality preprocessing compromises the embeddings

## Evidence

- [discussion] MUON modular design allows reuse of single-omics tools: "MUON is designed in a modular fashion, which means that existing methods and tools for processing individual omics can be reused"
- [methods] TF-IDF preprocessing for the ATAC modality: "TF-IDF transformation is implemented in the muon's ATAC module"
- [abstract] MUON as multi-omics framework: "we present a data standard and an analysis framework for multi-omics, MUON, designed to organise, analyse, visualise, and exchange multimodal data"
- [readme] muon as a Python framework: "`muon` is a multimodal omics Python framework."
