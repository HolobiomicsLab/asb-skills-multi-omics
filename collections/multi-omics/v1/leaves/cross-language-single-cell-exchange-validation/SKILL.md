---
name: cross-language-single-cell-exchange-validation
description: Use when a multimodal single-cell dataset stored as a .h5mu file (e.g.
  PBMC RNA+ATAC) must be loaded in R from a Python/Julia workflow, and cross-language
  fidelity of modalities and cell barcodes must be validated.
when_to_use_negative:
- The goal is Python-side analysis with muon itself (e.g. mu.pp.tfidf, mu.pp.filter_obs)
  rather than cross-language transfer to R
- The input is a raw format like 10x HDF5 or MTX (use muon.read_10x_h5 / read_10x_mtx)
  rather than an existing .h5mu container
- Only a single omics modality is present, where plain anndata/H5AD or SingleCellExperiment
  exchange suffices
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MUON
  role: defines the MuData .h5mu HDF5 format that makes multimodal data exchangeable
    across languages
  repo: https://github.com/scverse/muon
- name: MuData
  role: data structure serialised to HDF5 and read into R; MuData libraries exist
    for Python, R, and Julia
  repo: https://github.com/scverse/muon
- name: MultiAssayExperiment
  role: R container holding one experiment per modality in its ExperimentList with
    shared cell barcodes
- name: Seurat
  role: 'alternative R target: one assay per modality built from the same .h5mu stores'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:c928ca1e61216d6d9876db852846e2b66c3e9ed07f0698de14a14ade6df05e16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/muon
  tools:
  - MUON
  - MuData
  - MultiAssayExperiment
  - Seurat
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3076923076923077
n_claims: 13
---

# cross-language-single-cell-exchange-validation

## Summary

Reproducibly reads a MuData .h5mu file (HDF5 serialisation of multimodal omics data) into R objects (MultiAssayExperiment or Seurat), verifying that modality keys and cell counts are preserved across languages.

## When to use

Use when a multimodal single-cell dataset stored as a .h5mu file (e.g. PBMC RNA+ATAC) must be loaded in R from a Python/Julia workflow, and cross-language fidelity of modalities and cell barcodes must be validated.

## When NOT to use

- The goal is Python-side analysis with muon itself (e.g. mu.pp.tfidf, mu.pp.filter_obs) rather than cross-language transfer to R
- The input is a raw format like 10x HDF5 or MTX (use muon.read_10x_h5 / read_10x_mtx) rather than an existing .h5mu container
- Only a single omics modality is present, where plain anndata/H5AD or SingleCellExperiment exchange suffices

## Inputs

- MuData .h5mu file (HDF5 container with obs/obsm/var/varm groups and a .mod group of per-modality stores, e.g. PBMC RNA+ATAC)

## Outputs

- R MultiAssayExperiment with one experiment per modality ('rna', 'atac')
- optional Seurat object with one assay per modality
- serialised R object file mdata_pbmc.rds

## How to apply

Read the .h5mu file in R by traversing its HDF5 layout: top-level obs/obsm/var/varm groups plus the .mod group holding per-modality stores. Construct a MultiAssayExperiment whose ExperimentList contains one experiment per modality (e.g. 'rna' and 'atac'), mapping shared cell barcodes across modalities; optionally build an equivalent Seurat object with one assay per modality from the same .h5mu stores. Run a modality-content check verifying that modality keys ('atac', 'rna') and observation (cell) counts match the source container, exploiting that HDF5 serialisation makes MuData objects accessible to other programming languages including R and Julia. Serialise the in-memory R object with saveRDS (e.g. to mdata_pbmc.rds) so the exchange result is itself reloadable.

## Related tools

- **MUON** (defines the MuData .h5mu HDF5 format that makes multimodal data exchangeable across languages) — https://github.com/scverse/muon
- **MuData** (data structure serialised to HDF5 and read into R; MuData libraries exist for Python, R, and Julia) — https://github.com/scverse/muon
- **MultiAssayExperiment** (R container holding one experiment per modality in its ExperimentList with shared cell barcodes)
- **Seurat** (alternative R target: one assay per modality built from the same .h5mu stores)

## Examples

```
mdata <- readH5mu("pbmc.h5mu"); mae <- as(mdata, "MultiAssayExperiment"); stopifnot(all(names(mae) %in% c("rna","atac"))); saveRDS(mae, "mdata_pbmc.rds")
```

## Evaluation signals

- mdata_pbmc.rds exists and reloads in R with readRDS without error
- Loaded R object contains both expected modality keys ('atac' and 'rna')
- Observation (cell) counts in the R object equal the cell counts in the source .h5mu container
- Shared cell barcodes are consistently mapped across modalities in the ExperimentList
- Top-level HDF5 groups (obs, obsm, var, varm, .mod) were all traversed, so no modality store was silently dropped

## Limitations

- Support for emerging multi-omics assays (e.g. genomic-coordinate based or spatial-coordinate assays) is future work for MUON, so such modalities may not round-trip cleanly
- Explicit modelling of dependencies between feature sets across omics in MuData is planned future development, so cross-language objects cannot capture those relationships

## Evidence

- [discussion] HDF5 cross-language access: "Serialisation to HDF5 makes MuData objects accessible to other programming languages, including R and Julia."
- [intro] no prior exchange format: "Additionally, there currently exists no open exchange format for sharing multi-omics datasets that is accessible from different programming languages."
- [readme] muon framework purpose: "`muon` is a multimodal omics Python framework."
- [discussion] modular design rationale: "MUON is designed in a modular fashion, which means that existing methods and tools for processing individual omics can be reused to design more complex analysis workflows"
