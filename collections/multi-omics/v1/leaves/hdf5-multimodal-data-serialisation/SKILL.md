---
name: hdf5-multimodal-data-serialisation
description: Use when a multi-modality MuData object (e.g. paired 'rna'/'atac' or
  'rna'/'prot' PBMC assays) must be saved to or loaded from an .h5mu file, especially
  when analysis spans Python, R, or Julia.
when_to_use_negative:
- The dataset is a single-omics object (plain AnnData, Seurat, or SingleCellExperiment)
  with no modalities to coordinate — an ordinary .h5ad/RDS serialisation suffices.
- The target environment cannot read HDF5-backed MuData stores, or downstream tools
  expect a different container (e.g. flat CSV/MTX exports).
edam_operation: http://edamontology.org/operation_2406
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MUON
  role: Python framework that reads and writes MuData objects serialised to HDF5 (.h5mu),
    making them accessible from other languages
  repo: github.com/scverse/muon
- name: MuData
  role: Defines the HDF5 file format (top-level obs/obsm/var/varm plus .mod group)
    that the .h5mu stores follow
- name: MultiAssayExperiment
  role: R container built from the .h5mu layout, with one experiment per modality
    and shared cell barcodes mapped across modalities
- name: Seurat
  role: 'Alternative R representation: one assay per modality built from the same
    .h5mu stores'
merged_aliases:
- hdf5-multimodal-container-interoperability
merged_alias_records:
- alias: hdf5-multimodal-container-interoperability
  slug: hdf5-multimodal-container-interoperability
  cosine_score: 0.8342
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:a835d6ce82604f3332611fe2ca4322b1a26f57b59641379c69adf766e1a1aaee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/muon
  tools:
  - MUON
  - MuData
  - MultiAssayExperiment
  - NumPy
  - SCANPY
  - Seurat
  - pandas
  edam_operation: http://edamontology.org/operation_2406
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# HDF5 multimodal data serialisation

## Summary

Persist a MuData multimodal omics container to the HDF5-backed .h5mu format and read it back from other languages (R, Julia), enabling cross-language exchange of multi-omics data that previously had no open exchange format.

## When to use

Use when a multi-modality MuData object (e.g. paired 'rna'/'atac' or 'rna'/'prot' PBMC assays) must be saved to or loaded from an .h5mu file, especially when analysis spans Python, R, or Julia.

## When NOT to use

- The dataset is a single-omics object (plain AnnData, Seurat, or SingleCellExperiment) with no modalities to coordinate — an ordinary .h5ad/RDS serialisation suffices.
- The target environment cannot read HDF5-backed MuData stores, or downstream tools expect a different container (e.g. flat CSV/MTX exports).

## Inputs

- MuData multimodal omics container with per-modality AnnData stores (e.g. 'rna', 'atac', 'prot') sharing cell barcodes
- Existing .h5mu file following the MuData HDF5 layout (top-level obs/obsm/var/varm groups plus a .mod group)

## Outputs

- .h5mu HDF5 file with top-level obs/obsm/var/varm groups and a .mod group containing one store per modality
- R MultiAssayExperiment with an ExperimentList of one experiment per modality
- Optional Seurat object with one assay per modality
- Serialised R object such as mdata_pbmc.rds (saveRDS)

## How to apply

Serialise the in-memory MuData container to HDF5 with muon's write method, producing an .h5mu file whose layout keeps top-level obs/obsm/var/varm groups plus a .mod group holding one store per modality (e.g. 'rna', 'atac', 'prot'). To read in R, traverse this HDF5 layout directly and construct a MultiAssayExperiment whose ExperimentList holds one experiment per modality, mapping the shared cell barcodes across modalities; optionally build an equivalent Seurat object with one assay per modality from the same .h5mu stores. Before persisting R-side, run a modality-content check verifying that modality keys and observation (cell) counts match the source container. Finally serialise the in-memory R object with saveRDS (e.g. mdata_pbmc.rds) so it can be reused without re-reading HDF5. The rationale is that HDF5 serialisation is what makes MuData objects portable to R and Julia, closing the gap of no open, language-accessible exchange format for multi-omics datasets.

## Related tools

- **MUON** (Python framework that reads and writes MuData objects serialised to HDF5 (.h5mu), making them accessible from other languages) — github.com/scverse/muon
- **MuData** (Defines the HDF5 file format (top-level obs/obsm/var/varm plus .mod group) that the .h5mu stores follow)
- **MultiAssayExperiment** (R container built from the .h5mu layout, with one experiment per modality and shared cell barcodes mapped across modalities)
- **Seurat** (Alternative R representation: one assay per modality built from the same .h5mu stores)

## Examples

```
mdata.write("mdata_pbmc.h5mu")  # then in R: saveRDS(mdata_pbmc, "mdata_pbmc.rds")
```

## Evaluation signals

- The output .h5mu file exists and its HDF5 layout contains the top-level obs/obsm/var/varm groups plus the .mod group with per-modality stores.
- Modality keys (e.g. 'atac', 'rna') in the loaded object match the keys in the source container.
- Observation (cell) counts in each modality of the reloaded object equal the source .h5mu counts.
- The saved RDS (mdata_pbmc.rds) reloads in R without error and contains both expected modalities with cell counts equal to the source .h5mu.

## Limitations

- The exchange relies on the MuData HDF5 layout conventions (obs/obsm/var/varm and .mod groups); readers must traverse this layout to map modalities and barcodes correctly.
- The current MuData format does not yet explicitly model dependencies between feature sets across omics or between multiple sets of multi-omics experiments — such relationships are planned future development.
- Support for emerging multi-omics assays (e.g. genomic-coordinate based or spatial-coordinate assays) is future work, so such data may not serialise cleanly.

## Evidence

- [discussion] Serialisation to HDF5 makes MuData objects accessible to other programming languages, including R and Julia.: "Serialisation to HDF5 makes MuData objects accessible to other programming languages, including R and Julia."
- [intro] no open exchange format accessible from multiple languages: "Additionally, there currently exists no open exchange format for sharing multi-omics datasets that is accessible from different programming languages."
- [readme] muon is a multimodal omics Python framework: "`muon` is a multimodal omics Python framework."
- [abstract] MUON is a data standard and analysis framework for multi-omics designed to organise, analyse, visualise, and exchange multimodal data.: "we present a data standard and an analysis framework for multi-omics, MUON, designed to organise, analyse, visualise, and exchange multimodal data"
