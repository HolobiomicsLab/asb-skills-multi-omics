---
name: multimodal-factor-analysis
description: Use when paired multi-omics single-cell data (e.g. 10x PBMC Chromium
  Single Cell Multiome ATAC + Gene Expression in filtered_feature_bc_matrix.h5) needs
  unsupervised integration into shared latent factors spanning both RNA and ATAC modalities.
when_to_use_negative:
- Data are unimodal single-omics (e.g. scRNA-seq alone), where single-omics factor
  analysis or scanpy workflows suffice
- Modality-specific preprocessing such as ADT denoising (dsb) or TF-IDF/binarisation
  of ATAC peak counts is the goal — those are muon.pp steps, not factor analysis
- A supervised or covariate-aware model is needed beyond what default mu.tl.mofa provides;
  temporally aware factor analysis with covariates is described as a growing capability,
  not the default path
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3168
tools:
- name: MUON
  role: Provides the MuData data structure, mu.read_10x_h5 ingestion, and the mu.tl.mofa
    wrapper that trains MOFA+ across modalities
  repo: https://github.com/scverse/muon
- name: MOFA+
  role: Statistical framework that performs the multi-modal factor analysis learned
    inside mu.tl.mofa
- name: SCANPY
  role: Performs per-modality normalisation and scaling (sc.pp.normalise_per_cell
    with counts_per_cell_after=1e4, sc.pp.log1p) prior to factor analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:95102fa609acc5e659023e0e0d576b847716e8253048742bb91f431259d1405b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/muon
  tools:
  - MOFA+
  - MUON
  - SCANPY
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# multimodal-factor-analysis

## Summary

Jointly learns an interpretable latent factor space across paired single-cell modalities (e.g. scRNA-seq and scATAC-seq) by running MOFA+ factor analysis through muon's wrapper on a MuData object, capturing axes of biological variation such as the myeloid–lymphoid axis and cytotoxicity.

## When to use

Use when paired multi-omics single-cell data (e.g. 10x PBMC Chromium Single Cell Multiome ATAC + Gene Expression in filtered_feature_bc_matrix.h5) needs unsupervised integration into shared latent factors spanning both RNA and ATAC modalities.

## When NOT to use

- Data are unimodal single-omics (e.g. scRNA-seq alone), where single-omics factor analysis or scanpy workflows suffice
- Modality-specific preprocessing such as ADT denoising (dsb) or TF-IDF/binarisation of ATAC peak counts is the goal — those are muon.pp steps, not factor analysis
- A supervised or covariate-aware model is needed beyond what default mu.tl.mofa provides; temporally aware factor analysis with covariates is described as a growing capability, not the default path

## Inputs

- 10x multiome filtered feature-barcode matrix (filtered_feature_bc_matrix.h5) with paired 'rna' and 'atac' modalities
- MuData object with per-modality normalised, scaled matrices and a defined 'highly_variable' feature column

## Outputs

- MuData object with factors stored in mdata.obsm['X_mofa'] and factor loadings alongside
- Serialised multimodal object file 'mofa_pbmc_multiome.h5mu' (HDF5/MuData format)
- Plots of marker gene expression (RNA) and promoter accessibility (ATAC) along leading factors

## How to apply

Read the 10x multiome filtered feature-barcode matrix with mu.read_10x_h5 into a MuData object containing 'rna' and 'atac' modalities. Normalise each modality with log-normalisation (sc.pp.normalise_per_cell with counts_per_cell_after=1e4 followed by sc.pp.log1p for RNA, analogous log-normalisation for ATAC) and scale features. Ensure the 'highly_variable' feature-selection column is defined, since muon's MOFA wrapper trains by default on use_var='highly_variable'. Run mu.tl.mofa(mdata) to learn factors jointly across modalities, then plot marker gene expression (RNA) and promoter accessibility (ATAC) along the leading factors to interpret captured axes of variation, and persist the result with mdata.write('mofa_pbmc_multiome.h5mu').

## Related tools

- **MUON** (Provides the MuData data structure, mu.read_10x_h5 ingestion, and the mu.tl.mofa wrapper that trains MOFA+ across modalities) — https://github.com/scverse/muon
- **MOFA+** (Statistical framework that performs the multi-modal factor analysis learned inside mu.tl.mofa)
- **SCANPY** (Performs per-modality normalisation and scaling (sc.pp.normalise_per_cell with counts_per_cell_after=1e4, sc.pp.log1p) prior to factor analysis)

## Examples

```
import muon as mu, scanpy as sc; mdata = mu.read_10x_h5('filtered_feature_bc_matrix.h5'); sc.pp.normalise_per_cell(mdata['rna'], counts_per_cell_after=1e4); sc.pp.log1p(mdata['rna']); mu.tl.mofa(mdata); mdata.write('mofa_pbmc_multiome.h5mu')
```

## Evaluation signals

- "X_mofa" in mdata.obsm evaluates to True after mu.tl.mofa completes, confirming factors were stored
- Leading factors reproduce canonical biological axes: myeloid–lymphoid axis and cytotoxicity, visible in marker gene expression along factors
- Factor loadings are stored alongside the factors for both RNA and ATAC modalities
- mdata.write('mofa_pbmc_multiome.h5mu') succeeds, producing a valid MuData/HDF5 file

## Limitations

- muon's MOFA wrapper trains on use_var='highly_variable' by default, so results depend on the highly_variable column being correctly defined beforehand
- Support for emerging multi-omics assays and explicit modelling of feature-set dependencies across omics is future work for MUON, not current capability
- Explicit support for genomic-coordinate based assays and assays with spatial coordinates is listed as future development

## Evidence

- [full_text] MUON wrapper default training on highly variable features: [evidence span withheld — anchor status: not_found]
- [full_text] MOFA factors capture biological variation from both modalities: [evidence span withheld — anchor status: not_found]
- [abstract] MUON purpose as a multi-omics framework: "we present a data standard and an analysis framework for multi-omics, MUON, designed to organise, analyse, visualise, and exchange multimodal data"
- [discussion] HDF5 interoperability of MuData objects: "Serialisation to HDF5 makes MuData objects accessible to other programming languages, including R and Julia."
- [readme] muon is a multimodal omics Python framework: "`muon` is a multimodal omics Python framework."
