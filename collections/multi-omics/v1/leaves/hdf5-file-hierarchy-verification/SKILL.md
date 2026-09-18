---
name: hdf5-file-hierarchy-verification
description: Use when a MuData object has been written to .h5mu and you must confirm
  the HDF5 layout exposes per-modality AnnData storage, or when reading a single modality
  back (e.g. via mu.read('file.h5mu/rna')) to validate interoperable serialisation.
when_to_use_negative:
- Input is a raw single-modality file (e.g. a plain .h5ad or 10x mtx directory) with
  no multimodal container to verify
- The multimodal data must be exchanged in a format other than the MuData/HDF5 scheme,
  or the analysis requires explicit modelling of dependencies between feature sets
  across omics, which MuData does not yet support
tools:
- name: MUON
  role: Assembles the MuData container, writes it to .h5mu, and dispatches mu.read
    between read_h5mu and read_h5ad to reload individual modalities
  repo: github.com/scverse/muon
- name: SCANPY
  role: AnnData single-omics toolchain used to access individual modalities stored
    inside the HDF5 container
- name: NumPy
  role: Array backend underlying the AnnData/MuData data structures being serialised
- name: pandas
  role: DataFrame backend for the .obs/.var annotation tables synced by mdata.update()
    and stored in the HDF5 hierarchy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:63337a64f9521dc57eca73617ec147940e8884efc9bb8389ea98f09199dd8b05
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/muon
  tools:
  - MUON
  - NumPy
  - SCANPY
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# hdf5-file-hierarchy-verification

## Summary

Verifies that a MuData multimodal container serialised to .h5mu (HDF5) exhibits the correct hierarchy — a top-level 'mod' group holding per-modality AnnData groups (e.g. atac, rna) alongside obs/obsm/var/varm — so each omics layer remains readable with standard AnnData toolchains.

## When to use

Use when a MuData object has been written to .h5mu and you must confirm the HDF5 layout exposes per-modality AnnData storage, or when reading a single modality back (e.g. via mu.read('file.h5mu/rna')) to validate interoperable serialisation.

## When NOT to use

- Input is a raw single-modality file (e.g. a plain .h5ad or 10x mtx directory) with no multimodal container to verify
- The multimodal data must be exchanged in a format other than the MuData/HDF5 scheme, or the analysis requires explicit modelling of dependencies between feature sets across omics, which MuData does not yet support

## Inputs

- MuData object with AnnData modalities in .mod (e.g. {'rna': adata_rna, 'atac': adata_atac})
- Serialised .h5mu file (HDF5)
- Cell Ranger filtered_feature_bc_matrix.h5 file (via mu.read_10x_h5)
- Directory with matrix.mtx, features.tsv.gz, barcodes.tsv.gz (via mu.read_10x_mtx)
- Snap single-nucleus accessibility file (via ac.read_snap with matrix='bins', bin_size=10000)

## Outputs

- h5ls listing of mudata.h5mu showing mod group with atac and rna subgroups plus obs/obsm/var/varm
- AnnData object recovered from a modality path (e.g. mu.read('mudata.h5mu/rna'))
- Backed copy file filename_copy.h5mu (when copying a backed object)

## How to apply

After assembling the container with mu.MuData({'rna': adata_rna, 'atac': adata_atac}) and calling mdata.update() to sync joint .obs/.var annotations (modality columns gain prefixes like rna:n_genes), serialise with mdata.write('mudata.h5mu'). Inspect the HDF5 hierarchy with h5ls to confirm a top-level 'mod' Group containing per-modality Groups (atac, rna) next to obs/obsm/var/varm groups. Then read one modality back with mu.read('mudata.h5mu/rna') to confirm AnnData-compliant storage; mu.read dispatches between read_h5mu and read_h5ad automatically. If the object was opened with backed=True, verify copies with mdata.copy('filename_copy.h5mu'). The rationale is that HDF5 serialisation makes MuData objects accessible from other languages (R, Julia) and lets existing single-omics tools operate directly on each modality.

## Related tools

- **MUON** (Assembles the MuData container, writes it to .h5mu, and dispatches mu.read between read_h5mu and read_h5ad to reload individual modalities) — github.com/scverse/muon
- **SCANPY** (AnnData single-omics toolchain used to access individual modalities stored inside the HDF5 container)
- **NumPy** (Array backend underlying the AnnData/MuData data structures being serialised)
- **pandas** (DataFrame backend for the .obs/.var annotation tables synced by mdata.update() and stored in the HDF5 hierarchy)

## Examples

```
mdata = mu.MuData({'rna': adata_rna, 'atac': adata_atac}); mdata.update(); mdata.write('mudata.h5mu'); adata_rna = mu.read('mudata.h5mu/rna')
```

## Evaluation signals

- h5ls listing of mudata.h5mu must expose the top-level mod group with atac and rna subgroups
- mu.read('mudata.h5mu/rna') must return an AnnData object without error
- Joint .obs columns copied by mdata.update() carry modality-name prefixes (e.g. rna:n_genes)
- Identically named .var columns are merged across modalities after mdata.update()
- Serialised file is loadable by other languages (R, Julia) via the HDF5 format

## Limitations

- Explicitly modelling dependencies between feature sets across omics in MuData is planned for future development, so the container cannot encode cross-omics feature relationships
- Explicit support for genomic-coordinate based assays and assays with spatial coordinates is listed as future development
- Backing/copy semantics apply only when the object was read with backed=True

## Evidence

- [discussion] HDF5 serialisation makes MuData objects accessible from other programming languages: "Serialisation to HDF5 makes MuData objects accessible to other programming languages, including R and Julia."
- [abstract] MUON stores multimodal data in an efficient yet flexible and interoperable data structure: "MUON stores multimodal data in an efficient yet flexible and interoperable data structure."
- [discussion] MUON modular design allows existing single-omics methods to be reused: "MUON is designed in a modular fashion, which means that existing methods and tools for processing individual omics can be reused to design more complex analysis workflows"
- [readme] muon is a multimodal omics Python framework: "`muon` is a multimodal omics Python framework."
- [intro] no open exchange format for sharing multi-omics datasets: "Additionally, there currently exists no open exchange format for sharing multi-omics datasets that is accessible from different programming languages."
