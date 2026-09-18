---
name: multimodal-container-serialization
description: Use when single-cell count matrices from multiple modalities (e.g. RNA
  + ATAC from Cell Ranger outputs or a snap file) must be combined into one interoperable
  file and exchanged across languages or tools.
when_to_use_negative:
- Data is a single omics layer only — a plain AnnData .h5ad serialisation suffices
  and MuData adds no hierarchy
- You need explicitly modelled dependencies between feature sets across omics — the
  paper lists this as future development, not a current capability
- Assays with spatial or genomic-coordinate structure requiring explicit support —
  listed as planned future work in MUON
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_3170
tools:
- name: MUON
  role: Provides MuData container construction (mu.MuData, mdata.update) and HDF5
    .h5mu serialisation (mdata.write, mu.read with dispatch between read_h5mu and
    read_h5ad)
  repo: https://github.com/scverse/muon
- name: SCANPY
  role: Underlying single-cell AnnData ecosystem whose AnnData serialisation format
    is reused for each modality inside the .h5mu file
- name: NumPy
  role: Array backing for the count matrices held in the container
- name: pandas
  role: DataFrame backing for joint and modality-specific .obs/.var metadata annotations
- name: SnapATAC
  role: Source of snap files describing single-nucleus accessibility profiles, ingested
    per modality via ac.read_snap
  repo: https://github.com/r3fang/SnapATAC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:c9d30b05055a1f934b75909affcd94724f12e707d6a88c27387ec03614781ca8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/muon
  tools:
  - MUON
  - NumPy
  - SCANPY
  - SnapATAC
  - pandas
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: thin
yes_rate: 0.0
n_claims: 0
---

# multimodal-container-serialization

## Summary

Assemble per-modality AnnData objects into a MuData container and serialise it to a single .h5mu (HDF5) file, so each omics layer remains independently accessible with existing AnnData toolchains while joint metadata is shared.

## When to use

Use when single-cell count matrices from multiple modalities (e.g. RNA + ATAC from Cell Ranger outputs or a snap file) must be combined into one interoperable file and exchanged across languages or tools.

## When NOT to use

- Data is a single omics layer only — a plain AnnData .h5ad serialisation suffices and MuData adds no hierarchy
- You need explicitly modelled dependencies between feature sets across omics — the paper lists this as future development, not a current capability
- Assays with spatial or genomic-coordinate structure requiring explicit support — listed as planned future work in MUON

## Inputs

- Cell Ranger filtered_feature_bc_matrix.h5 file (10x Genomics)
- 10x mtx directory with matrix.mtx, features.tsv.gz, barcodes.tsv.gz
- SnapATAC .snap file (single-nucleus accessibility profiles)
- AnnData objects, one per modality (e.g. adata_rna, adata_atac)

## Outputs

- MuData object with modalities stored in the .mod attribute
- mudata.h5mu file: HDF5 with a top-level 'mod' group (per-modality AnnData groups, e.g. atac, rna) alongside obs/obsm/var/varm groups

## How to apply

Ingest each omics layer as an AnnData object with muon (mu.read_10x_h5 on a filtered_feature_bc_matrix.h5, mu.read_10x_mtx on matrix.mtx/features.tsv.gz/barcodes.tsv.gz directories, or ac.read_snap with matrix='bins' and bin_size=10000 for a snap file). Assemble with mu.MuData({'rna': adata_rna, 'atac': adata_atac}) so modalities live in the .mod attribute keyed by name, then call mdata.update() to synchronise joint .obs/.var annotations — modality columns are copied with name prefixes (e.g. rna:n_genes) and identically named .var columns are merged across modalities. Serialise with mdata.write('mudata.h5mu'); HDF5 serialisation is what makes MuData objects accessible from other languages such as R and Julia. Confirm the layout by inspecting the HDF5 hierarchy (top-level mod group with per-modality groups alongside obs/obsm/var/varm) and by reading a single modality back with mu.read('mudata.h5mu/rna').

## Related tools

- **MUON** (Provides MuData container construction (mu.MuData, mdata.update) and HDF5 .h5mu serialisation (mdata.write, mu.read with dispatch between read_h5mu and read_h5ad)) — https://github.com/scverse/muon
- **SCANPY** (Underlying single-cell AnnData ecosystem whose AnnData serialisation format is reused for each modality inside the .h5mu file)
- **NumPy** (Array backing for the count matrices held in the container)
- **pandas** (DataFrame backing for joint and modality-specific .obs/.var metadata annotations)
- **SnapATAC** (Source of snap files describing single-nucleus accessibility profiles, ingested per modality via ac.read_snap) — https://github.com/r3fang/SnapATAC

## Examples

```
mdata = mu.MuData({'rna': adata_rna, 'atac': adata_atac}); mdata.update(); mdata.write('mudata.h5mu')
```

## Evaluation signals

- h5ls listing of mudata.h5mu exposes a top-level 'mod' group containing per-modality subgroups (e.g. atac, rna) alongside obs/obsm/var/varm groups
- mu.read('mudata.h5mu/rna') returns an AnnData object without error, confirming AnnData-compliant per-modality storage
- After mdata.update(), joint .obs contains modality-prefixed columns (e.g. rna:n_genes) copied from modality annotations
- mu.read dispatches correctly between read_h5mu and read_h5ad for the given path
- A backed copy via mdata.copy('filename_copy.h5mu') succeeds when the object was read with backed=True

## Limitations

- Explicit support for genomic-coordinate based assays and assays with spatial coordinates is future development, not current capability
- Explicitly modelling dependencies between feature sets across omics in MuData, and dependencies between multiple sets of multi-omics experiments, is planned future work
- Support for emerging multi-omics assays is identified as future work rather than a current capability

## Evidence

- [methods] In the MuData serialisation scheme, each individual omics layer is stored as an AnnData object using the existing AnnData serialisation format inside an HDF5 file: [evidence span withheld — anchor status: not_found]
- [discussion] Serialisation to HDF5 makes MuData objects accessible to other programming languages, including R and Julia.: "Serialisation to HDF5 makes MuData objects accessible to other programming languages, including R and Julia."
- [abstract] MUON stores multimodal data in an efficient yet flexible and interoperable data structure.: "MUON stores multimodal data in an efficient yet flexible and interoperable data structure"
- [readme] `muon` is a multimodal omics Python framework.: "`muon` is a multimodal omics Python framework."
- [methods] atac = ac.read_snap("pbmc3k.snap", matrix="bins", bin_size=10000): "atac = ac.read_snap("pbmc3k.snap", matrix="bins", bin_size=10000)"
