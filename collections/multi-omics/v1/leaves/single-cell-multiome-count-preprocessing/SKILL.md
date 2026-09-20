---
name: single-cell-multiome-count-preprocessing
description: Use when starting from raw 10x Genomics multiome output (e.g. filtered_feature_bc_matrix.h5)
  that must be normalised per modality and feature-selected before multi-omics factor
  analysis or alignment.
when_to_use_negative:
- The dataset is a CITE-seq / protein-tag modality requiring antibody-derived-tag
  denoising (dsb) or CLR normalisation rather than RNA/ATAC log-normalisation
- The input is already a normalised MuData/AnnData object with 'highly_variable' defined
  — re-running log-normalisation (1e4 target counts + log1p) and TF-IDF would double-normalise
  the counts
- The goal is single-omics analysis of one assay only, where the multimodal MuData
  container and cross-modality steps (intersect_obs) add no value
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MUON
  role: 'Core framework: reads 10x h5/mtx multiome inputs, holds the ''rna'' and ''atac''
    modalities in a MuData object, provides in-place filtering (filter_obs/filter_var,
    intersect_obs) and the ATAC-module TF-IDF transformation'
  repo: https://github.com/scverse/muon
- name: SCANPY
  role: 'Per-modality RNA preprocessing: sc.pp.normalise_per_cell with counts_per_cell_after=1e4,
    sc.pp.log1p, feature scaling'
- name: MOFA+
  role: 'Downstream consumer of the preprocessed object: muon''s MOFA wrapper trains
    on features flagged ''highly_variable'', so preprocessing must define that column'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:14a41e7ffe9d23bb46b76e3248762043e54208a2339483eb3ac5e96a51619145
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/muon
  tools:
  - MOFA+
  - MUON
  - SCANPY
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-multiome-count-preprocessing

## Summary

Preprocess paired single-cell multiome count matrices (scRNA-seq plus scATAC-seq) into a normalised, feature-selected MuData object ready for joint multimodal factor analysis, using the MUON framework with Scanpy. This conditioning on a shared cell-by-cell structure is what makes downstream MOFA integration over the 'rna' and 'atac' modalities possible.

## When to use

Use when starting from raw 10x Genomics multiome output (e.g. filtered_feature_bc_matrix.h5) that must be normalised per modality and feature-selected before multi-omics factor analysis or alignment.

## When NOT to use

- The dataset is a CITE-seq / protein-tag modality requiring antibody-derived-tag denoising (dsb) or CLR normalisation rather than RNA/ATAC log-normalisation
- The input is already a normalised MuData/AnnData object with 'highly_variable' defined — re-running log-normalisation (1e4 target counts + log1p) and TF-IDF would double-normalise the counts
- The goal is single-omics analysis of one assay only, where the multimodal MuData container and cross-modality steps (intersect_obs) add no value

## Inputs

- 10x Genomics multiome filtered feature-barcode matrix file (filtered_feature_bc_matrix.h5) with paired scRNA-seq and scATAC-seq modalities
- Optionally matrix.mtx / features.tsv.gz / barcodes.tsv.gz directories readable via mu.read_10x_mtx

## Outputs

- Normalised, feature-selected MuData object with 'rna' and 'atac' modalities and a 'highly_variable' column per modality
- HDF5-serialised MuData file (e.g. mofa_pbmc_multiome.h5mu) exchangeable with other programming languages

## How to apply

Read the 10x filtered feature-barcode matrix (filtered_feature_bc_matrix.h5) with mu.read_10x_h5 into a MuData object containing 'rna' and 'atac' modalities sharing the same cell barcodes. Normalise the RNA modality with Scanpy log-normalisation (sc.pp.normalise_per_cell with counts_per_cell_after=1e4 followed by sc.pp.log1p) and scale features; apply the analogous log-normalisation to the ATAC modality, optionally with TF-IDF (ac.pp.tfidf with scale_factor=1e4) since it is implemented in muon's ATAC module. Ensure a 'highly_variable' feature-selection column is defined in each modality, because muon's MOFA wrapper defaults to training on use_var='highly_variable'. Use muon's in-place filtering helpers (mu.pp.filter_obs / mu.pp.filter_var) and mu.pp.intersect_obs to keep observations present in all modalities, then write the result as HDF5 with mdata.write('mofa_pbmc_multiome.h5mu'). The rationale is MUON's modular design: existing single-omics preprocessing tools are reused inside a multimodal data structure before integration.

## Related tools

- **MUON** (Core framework: reads 10x h5/mtx multiome inputs, holds the 'rna' and 'atac' modalities in a MuData object, provides in-place filtering (filter_obs/filter_var, intersect_obs) and the ATAC-module TF-IDF transformation) — https://github.com/scverse/muon
- **SCANPY** (Per-modality RNA preprocessing: sc.pp.normalise_per_cell with counts_per_cell_after=1e4, sc.pp.log1p, feature scaling)
- **MOFA+** (Downstream consumer of the preprocessed object: muon's MOFA wrapper trains on features flagged 'highly_variable', so preprocessing must define that column)

## Examples

```
mdata = mu.read_10x_h5('filtered_feature_bc_matrix.h5'); ac.pp.tfidf(mdata['atac'], scale_factor=1e4); mdata.write('mofa_pbmc_multiome.h5mu')
```

## Evaluation signals

- MuData object contains both modalities named 'rna' and 'atac' with identical cell-barcode observations (verify with mu.pp.intersect_obs if needed)
- RNA modality counts are log-normalised at counts_per_cell_after=1e4 with log1p applied (non-integer values, no raw integer counts remaining)
- A boolean 'highly_variable' column exists in both modalities, since muon's MOFA wrapper defaults to use_var='highly_variable'
- The output serialises to HDF5 and round-trips via mdata.write('*.h5mu') / re-reading
- Downstream mu.tl.mofa runs successfully and 'X_mofa' in mdata.obsm evaluates to True

## Limitations

- Support for emerging multi-omics assays, genomic-coordinate based assays, and assays with spatial coordinates is future work for MUON, not a current capability of this preprocessing path
- If no 'highly_variable' column is defined, muon's default MOFA training (use_var='highly_variable') cannot proceed, so feature selection is a hard prerequisite
- Explicitly modelling dependencies between feature sets across omics within MuData is planned for future development; current preprocessing treats modalities independently until integration

## Evidence

- [abstract] MUON abstract framework quote: "we present a data standard and an analysis framework for multi-omics, MUON, designed to organise, analyse, visualise, and exchange multimodal data"
- [methods] muon.read_10x_mtx directory formats: "{func}`muon.read_10x_mtx` methods work in the same way for directories with files `matrix.mtx`, `features.tsv.gz`, and `barcodes.tsv.gz`."
- [methods] ATAC TF-IDF transformation: "TF-IDF transformation is implemented in the muon's ATAC module"
- [methods] in-place filtering functions: "`muon` introduces functions for in-place filtering: {func}`muon.pp.filter_obs` and {func}`muon.pp.filter_var`"
- [discussion] HDF5 cross-language serialisation: "Serialisation to HDF5 makes MuData objects accessible to other programming languages, including R and Julia."
- [readme] muon README descriptor: "`muon` is a multimodal omics Python framework."
