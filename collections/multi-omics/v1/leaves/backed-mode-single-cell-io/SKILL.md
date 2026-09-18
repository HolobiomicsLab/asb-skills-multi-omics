---
name: backed-mode-single-cell-io
description: Use when a multimodal (e.g. RNA + ATAC) MuData container must be written
  to disk as .h5mu, re-opened backed for large data, or a single modality read back
  as AnnData.
when_to_use_negative:
- The data is a single-omics dataset with no modality structure — write it directly
  as .h5ad with AnnData instead
- You need an open exchange format beyond what HDF5-backed MuData supports, e.g. explicitly
  modelling dependencies between feature sets across omics, which is planned future
  development of MUON
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MUON
  role: Provides MuData container assembly (mu.MuData), mdata.update(), mdata.write
    to .h5mu, backed copy, and mu.read with automatic read_h5mu/read_h5ad dispatch
  repo: https://github.com/scverse/muon
- name: SCANPY
  role: AnnData serialisation format reused per modality inside the HDF5 file
- name: NumPy
  role: Array storage backend for the serialised matrices
- name: pandas
  role: Dataframe storage for .obs/.var annotations synced by mdata.update()
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:c2046edca83c115c309a154bb0503edff99ced8bf04808425aefc234256f753e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/muon
  tools:
  - MUON
  - NumPy
  - SCANPY
  - pandas
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# Backed-mode single-cell IO (MuData/AnnData HDF5 serialisation)

## Summary

Serialise multimodal omics containers to MuData's .h5mu HDF5 format, where each modality is stored as a standard AnnData object, so individual omics layers remain readable with existing single-cell toolchains (including from R and Julia) and large objects can be handled via backed mode.

## When to use

Use when a multimodal (e.g. RNA + ATAC) MuData container must be written to disk as .h5mu, re-opened backed for large data, or a single modality read back as AnnData.

## When NOT to use

- The data is a single-omics dataset with no modality structure — write it directly as .h5ad with AnnData instead
- You need an open exchange format beyond what HDF5-backed MuData supports, e.g. explicitly modelling dependencies between feature sets across omics, which is planned future development of MUON

## Inputs

- MuData container with modalities stored in .mod (e.g. {'rna': adata_rna, 'atac': adata_atac})
- AnnData objects ingested per modality (from mu.read_10x_h5, mu.read_10x_mtx, or ac.read_snap)
- Existing .h5mu file path or in-file path to a modality (e.g. mudata.h5mu/rna)

## Outputs

- mudata.h5mu HDF5 file with top-level mod group (atac, rna subgroups) plus obs/obsm/var/varm groups
- Backed-mode copy file (e.g. filename_copy.h5mu)
- AnnData object returned when reading a single modality path such as mudata.h5mu/rna

## How to apply

Assemble the container with mu.MuData({'rna': adata_rna, 'atac': adata_atac}) so modalities live under the .mod attribute, then call mdata.update() to sync joint .obs/.var annotations (modality-prefixed columns like rna:n_genes, identically named .var columns merged). Serialise with mdata.write('mudata.h5mu'); the HDF5 layout stores each modality as an AnnData object using the existing AnnData serialisation format inside a top-level mod group alongside obs/obsm/var/varm groups. When the object was read with backed=True, create a backed copy via mdata.copy('filename_copy.h5mu'). Read an individual modality back with mu.read('mudata.h5mu/rna'); mu.read dispatches automatically between read_h5mu and read_h5ad.

## Related tools

- **MUON** (Provides MuData container assembly (mu.MuData), mdata.update(), mdata.write to .h5mu, backed copy, and mu.read with automatic read_h5mu/read_h5ad dispatch) — https://github.com/scverse/muon
- **SCANPY** (AnnData serialisation format reused per modality inside the HDF5 file)
- **NumPy** (Array storage backend for the serialised matrices)
- **pandas** (Dataframe storage for .obs/.var annotations synced by mdata.update())

## Examples

```
import muon as mu; mdata = mu.MuData({'rna': adata_rna, 'atac': adata_atac}); mdata.update(); mdata.write('mudata.h5mu'); adata = mu.read('mudata.h5mu/rna')
```

## Evaluation signals

- h5ls listing of mudata.h5mu exposes a top-level mod Group containing per-modality Groups (atac, rna) alongside obs/obsm/var/varm groups
- mu.read('mudata.h5mu/rna') returns an AnnData object without error, confirming AnnData-compliant storage
- mdata.update() has copied modality columns with modality-name prefixes (e.g. rna:n_genes) into joint .obs and merged identically named .var columns
- mdata.copy('filename_copy.h5mu') succeeds when the object was read with backed=True
- The .h5mu file can be accessed from other programming languages, since serialisation to HDF5 makes MuData objects accessible to R and Julia

## Limitations

- Explicitly modeling dependencies between feature sets across omics in MuData is planned for future development, not a current capability of the serialisation scheme
- Handling novel emerging multi-omics assays, and explicit support for genomic-coordinate based assays or assays with spatial coordinates, is future work for MUON
- Backed copies are created via mdata.copy only when the object was originally read with backed=True

## Evidence

- [discussion] HDF5 serialisation makes MuData objects accessible from other programming languages: "Serialisation to HDF5 makes MuData objects accessible to other programming languages, including R and Julia."
- [methods] AnnData-compatible per-modality storage: [evidence span withheld — anchor status: not_found]
- [methods] Ingestion of snap files per modality: "atac = ac.read_snap("pbmc3k.snap", matrix="bins", bin_size=10000)"
- [readme] Muon framework scope: "`muon` is a multimodal omics Python framework."
