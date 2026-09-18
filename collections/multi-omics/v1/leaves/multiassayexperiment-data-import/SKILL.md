---
name: multiassayexperiment-data-import
description: Use when a multi-omics dataset stored as a MuData .h5mu file (e.g. PBMC
  RNA+ATAC) must be read into R as MultiAssayExperiment or Seurat objects, preserving
  shared cell barcodes across modalities.
when_to_use_negative:
- The data is already a native R object (SingleCellExperiment, Seurat, or MultiAssayExperiment)
  — no cross-language import is needed
- The file is a plain AnnData .h5ad or 10x HDF5 file rather than a multimodal .h5mu
  container; use muon.read_10x_h5 / read_10x_mtx or anndata readers instead
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_3174
tools:
- name: MUON / MuData
  role: Defines the MuData HDF5 (.h5mu) format and provides libraries to create, read,
    and write it from Python, R, and Julia, enabling the cross-language import this
    skill performs
  repo: https://github.com/scverse/muon
- name: MultiAssayExperiment
  role: R Bioconductor class used to rebuild the .h5mu container, with an ExperimentList
    holding one experiment per modality ('rna', 'atac') sharing cell barcodes
- name: Seurat
  role: 'Optional R representation: builds an equivalent object with one assay per
    modality from the same .h5mu stores'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:fe2cefe02e3348dc58a920e4a8c7f62695a963291fc81e1faba65b49e81789c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/muon
  tools:
  - MUON
  - MUON / MuData
  - MultiAssayExperiment
  - Seurat
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 12
---

# multiassayexperiment-data-import

## Summary

Imports a MUON MuData .h5mu container into R by traversing its HDF5 layout (obs/obsm/var/varm plus the .mod group of per-modality stores) and rebuilding it as a MultiAssayExperiment (and optionally a Seurat object), enabling cross-language exchange of multimodal omics data.

## When to use

Use when a multi-omics dataset stored as a MuData .h5mu file (e.g. PBMC RNA+ATAC) must be read into R as MultiAssayExperiment or Seurat objects, preserving shared cell barcodes across modalities.

## When NOT to use

- The data is already a native R object (SingleCellExperiment, Seurat, or MultiAssayExperiment) — no cross-language import is needed
- The file is a plain AnnData .h5ad or 10x HDF5 file rather than a multimodal .h5mu container; use muon.read_10x_h5 / read_10x_mtx or anndata readers instead

## Inputs

- MuData .h5mu file (HDF5 layout: obs/obsm/var/varm groups plus .mod group with per-modality stores such as 'rna' and 'atac')

## Outputs

- MultiAssayExperiment with an ExperimentList containing one experiment per modality, cell barcodes mapped across modalities
- Seurat object with one assay per modality (optional)
- Serialised R object file (e.g. mdata_pbmc.rds) written via saveRDS

## How to apply

Read the .h5mu file in R by traversing its HDF5 layout: the top-level obs/obsm/var/varm groups hold shared cell and feature metadata, while the .mod group holds one HDF5 store per modality (e.g. 'rna' and 'atac'). Build a MultiAssayExperiment whose ExperimentList contains one experiment per modality, mapping the shared cell barcodes from obs across modalities so experiments are sample-aligned. Optionally construct an equivalent Seurat object with one assay per modality from the same .mod stores. This works because MUON serialises MuData objects to HDF5, making them accessible from R and Julia as well as Python. Finally validate that the modality keys and cell counts match the source container, then serialise with saveRDS.

## Related tools

- **MUON / MuData** (Defines the MuData HDF5 (.h5mu) format and provides libraries to create, read, and write it from Python, R, and Julia, enabling the cross-language import this skill performs) — https://github.com/scverse/muon
- **MultiAssayExperiment** (R Bioconductor class used to rebuild the .h5mu container, with an ExperimentList holding one experiment per modality ('rna', 'atac') sharing cell barcodes)
- **Seurat** (Optional R representation: builds an equivalent object with one assay per modality from the same .h5mu stores)

## Examples

```
saveRDS(mdata, "mdata_pbmc.rds")
```

## Evaluation signals

- The serialised mdata_pbmc.rds file exists on disk and reloads in R without error via readRDS
- The loaded R object contains both expected modality keys ('atac' and 'rna')
- Cell (observation) counts in the imported MultiAssayExperiment equal the counts in the source .h5mu container
- Shared cell barcodes are consistent across modalities in the ExperimentList (experiments are sample-aligned)
- Per-modality content (feature metadata in var/varm, low-dimensional embeddings in obsm) matches the corresponding .mod stores in the HDF5 file

## Limitations

- Explicitly modelling dependencies between feature sets across omics, and accounting for dependencies between multiple sets of multi-omics experiments, is planned future MUON development, so the .h5mu format may not capture such relationships during import
- The import faithfully transfers the container structure but relies on the HDF5 serialisation convention; novel assay types with genomic or spatial coordinates are listed as future MUON support rather than current capability

## Evidence

- [discussion] Cross-language accessibility via HDF5: "Serialisation to HDF5 makes MuData objects accessible to other programming languages, including R and Julia."
- [intro] Motivation: no open cross-language exchange format: "Additionally, there currently exists no open exchange format for sharing multi-omics datasets that is accessible from different programming languages."
- [abstract] MUON purpose as exchange framework: "we present a data standard and an analysis framework for multi-omics, MUON, designed to organise, analyse, visualise, and exchange multimodal data"
- [readme] muon as the source of the .h5mu container: "`muon` is a multimodal omics Python framework."
