---
name: r-object-serialization
description: Use when a Python-created MuData .h5mu file must be consumed in R, or
  when a multi-modality R object (e.g. PBMC 'rna' + 'atac') needs to be saved to .rds
  for reuse.
when_to_use_negative:
- The data are single-modality AnnData/.h5ad files with no .mod group, so no multi-modality
  mapping is required
- The target consumer is Python/Julia rather than R — read or write the .h5mu directly
  via the muon/mudata libraries instead of round-tripping through .rds
- Only filtered cells/features or preprocessing (e.g. filter_obs, filter_var, TF-IDF)
  is needed, not cross-language exchange or persistence
edam_operation: http://edamontology.org/operation_2409
tools:
- name: MUON
  role: Defines the MuData (.h5mu) HDF5 data standard whose stores are read into R
  repo: https://github.com/scverse/muon
- name: MultiAssayExperiment
  role: R container constructed from the .h5mu stores; its ExperimentList holds one
    experiment per modality with shared cell barcodes mapped across modalities
- name: Seurat
  role: Optional alternative R representation with one assay per modality built from
    the same .h5mu stores
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:9c85530277ea051b75c4891c719f45bb189efa41a4b03db6fbec125d74067498
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/muon
  tools:
  - MUON
  - MultiAssayExperiment
  - Seurat
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# R object serialization

## Summary

Reads MuData .h5mu HDF5 stores into R (MultiAssayExperiment or Seurat) and serialises the resulting in-memory object with saveRDS, enabling cross-language exchange of multi-omics data via the MUON format.

## When to use

Use when a Python-created MuData .h5mu file must be consumed in R, or when a multi-modality R object (e.g. PBMC 'rna' + 'atac') needs to be saved to .rds for reuse.

## When NOT to use

- The data are single-modality AnnData/.h5ad files with no .mod group, so no multi-modality mapping is required
- The target consumer is Python/Julia rather than R — read or write the .h5mu directly via the muon/mudata libraries instead of round-tripping through .rds
- Only filtered cells/features or preprocessing (e.g. filter_obs, filter_var, TF-IDF) is needed, not cross-language exchange or persistence

## Inputs

- MuData .h5mu file (HDF5 with top-level obs/obsm/var/varm groups and a .mod group of per-modality stores)
- Source container metadata (modality keys such as 'rna'/'atac' and observation/cell counts) for validation

## Outputs

- MultiAssayExperiment object with an ExperimentList of one experiment per modality
- Optional Seurat object with one assay per modality
- mdata_pbmc.rds serialised R object

## How to apply

Read the PBMC .h5mu file in R by traversing its HDF5 layout: the top-level obs/obsm/var/varm groups plus the .mod group holding per-modality stores. Construct a MultiAssayExperiment whose ExperimentList contains one experiment per modality (e.g. 'rna' and 'atac'), mapping shared cell barcodes across modalities; optionally build an equivalent Seurat object with one assay per modality from the same .h5mu stores. Run a modality-content check that the modality keys ('atac', 'rna') and observation (cell) counts match the source container. Serialise the in-memory R object with saveRDS to mdata_pbmc.rds, then reload it to confirm it restores without error. The rationale is that MUON's HDF5 serialisation is the open exchange mechanism that makes MuData objects accessible from R, which was previously unavailable for multi-omics datasets.

## Related tools

- **MUON** (Defines the MuData (.h5mu) HDF5 data standard whose stores are read into R) — https://github.com/scverse/muon
- **MultiAssayExperiment** (R container constructed from the .h5mu stores; its ExperimentList holds one experiment per modality with shared cell barcodes mapped across modalities)
- **Seurat** (Optional alternative R representation with one assay per modality built from the same .h5mu stores)

## Examples

```
saveRDS(mdata_pbmc, "mdata_pbmc.rds"); mdata_reloaded <- readRDS("mdata_pbmc.rds")
```

## Evaluation signals

- mdata_pbmc.rds exists on disk after the saveRDS step
- readRDS reloads the file in R without error
- The reloaded object contains both expected modality keys ('atac' and 'rna')
- Cell (observation) counts per modality in the R object equal those in the source .h5mu container
- Shared cell barcodes map consistently across modalities in the ExperimentList

## Limitations

- HDF5 serialisation is required for cross-language access: a MuData object kept purely in Python memory cannot be consumed in R without writing the .h5mu first
- MUON's modular per-omics tools (e.g. ac.pp.tfidf, pt.pp.clr) operate in Python; equivalent processing in R is outside this skill's scope
- Support for assays with genomic/spatial coordinates or explicit cross-omics feature-set dependencies is planned future MUON development, so such modalities may not translate cleanly

## Evidence

- [discussion] HDF5 serialisation makes MuData objects accessible to other programming languages including R and Julia: "Serialisation to HDF5 makes MuData objects accessible to other programming languages, including R and Julia."
- [intro] no open exchange format across languages: "Additionally, there currently exists no open exchange format for sharing multi-omics datasets that is accessible from different programming languages."
- [readme] muon is a multimodal omics framework: "`muon` is a multimodal omics Python framework."
- [abstract] MUON is a data standard and analysis framework for multi-omics designed to organise, analyse, visualise, and exchange multimodal data: "we present a data standard and an analysis framework for multi-omics, MUON, designed to organise, analyse, visualise, and exchange multimodal data"
