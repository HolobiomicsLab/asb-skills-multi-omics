---
name: anndata-object-construction
description: Use when single-cell count matrices from multiple modalities (e.g. 10x
  filtered_feature_bc_matrix.h5, matrix.mtx directories, or .snap files) must be combined
  into one multimodal object with joint metadata.
when_to_use_negative:
- Input is already a single-modality feature table or processed matrix that does not
  need multimodal container assembly
- The analysis targets novel multi-omics assays or spatially coordinated assays that
  MUON explicitly lists as future work rather than current capability
- You only need to exchange single-omics data — a plain .h5ad AnnData file suffices
  without a MuData wrapper
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MUON
  role: Provides mu.read_10x_h5/mu.read_10x_mtx/ac.read_snap ingestion, the MuData
    container class, mdata.update() syncing, and .h5mu serialisation
  repo: github.com/scverse/muon
- name: SCANPY
  role: Underlying single-cell AnnData data structure and analysis functions reused
    per modality inside the MuData container
- name: SnapATAC
  role: Source of .snap single-nucleus accessibility profiles ingested via ac.read_snap
    with matrix='bins'
  repo: github.com/r3fang/SnapATAC
- name: NumPy
  role: Array layer underpinning AnnData matrix storage
- name: pandas
  role: DataFrame layer for .obs/.var annotation syncing during mdata.update()
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:0f7f3baedc1b03f791ebef661bb700ffbdc8c6c9c22fe5be8a62a9b31c64f84d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MUON
  - NumPy
  - SCANPY
  - SnapATAC
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# anndata-object-construction

## Summary

Constructs AnnData objects per omics modality and assembles them into a MuData multimodal container serialised to .h5mu (HDF5), so each single-omics layer remains AnnData-compliant and accessible with existing toolchains.

## When to use

Use when single-cell count matrices from multiple modalities (e.g. 10x filtered_feature_bc_matrix.h5, matrix.mtx directories, or .snap files) must be combined into one multimodal object with joint metadata.

## When NOT to use

- Input is already a single-modality feature table or processed matrix that does not need multimodal container assembly
- The analysis targets novel multi-omics assays or spatially coordinated assays that MUON explicitly lists as future work rather than current capability
- You only need to exchange single-omics data — a plain .h5ad AnnData file suffices without a MuData wrapper

## Inputs

- Cell Ranger filtered_feature_bc_matrix.h5 file (10x Genomics)
- 10x mtx directory with matrix.mtx, features.tsv.gz, barcodes.tsv.gz
- SnapATAC .snap single-nucleus accessibility file
- Per-modality AnnData objects (adata_rna, adata_atac)

## Outputs

- MuData container with modalities in .mod attribute keyed by name
- mudata.h5mu HDF5 file with top-level mod group containing per-modality groups (atac, rna) alongside obs/obsm/var/varm groups
- Synced joint .obs/.var annotations with modality-prefixed columns (e.g. rna:n_genes)

## How to apply

Ingest each modality into an AnnData object with muon readers: mu.read_10x_h5 on a Cell Ranger filtered_feature_bc_matrix.h5 file, mu.read_10x_mtx for directories with matrix.mtx/features.tsv.gz/barcodes.tsv.gz, or ac.read_snap('pbmc3k.snap', matrix='bins', bin_size=10000) for snap files. Assemble the container with mu.MuData({'rna': adata_rna, 'atac': adata_atac}) so modalities live in the .mod attribute keyed by name. Call mdata.update() to sync joint .obs/.var annotations — modality columns are copied with modality-name prefixes (e.g. rna:n_genes) and identically named .var columns are merged across modalities. Serialise with mdata.write('mudata.h5mu'); if the object was read with backed=True, create a backed copy via mdata.copy('filename_copy.h5mu'). The rationale is that each modality is stored using the existing AnnData serialisation format inside one HDF5 file, so single omics can be read back directly with existing toolchains.

## Related tools

- **MUON** (Provides mu.read_10x_h5/mu.read_10x_mtx/ac.read_snap ingestion, the MuData container class, mdata.update() syncing, and .h5mu serialisation) — github.com/scverse/muon
- **SCANPY** (Underlying single-cell AnnData data structure and analysis functions reused per modality inside the MuData container)
- **SnapATAC** (Source of .snap single-nucleus accessibility profiles ingested via ac.read_snap with matrix='bins') — github.com/r3fang/SnapATAC
- **NumPy** (Array layer underpinning AnnData matrix storage)
- **pandas** (DataFrame layer for .obs/.var annotation syncing during mdata.update())

## Examples

```
mdata = mu.MuData({'rna': mu.read_10x_h5('filtered_feature_bc_matrix.h5'), 'atac': ac.read_snap('pbmc3k.snap', matrix='bins', bin_size=10000)}); mdata.update(); mdata.write('mudata.h5mu')
```

## Evaluation signals

- h5ls listing of mudata.h5mu must expose a top-level 'mod' group containing 'atac' and 'rna' subgroups alongside obs/obsm/var/varm groups
- mu.read('mudata.h5mu/rna') must return an AnnData object without error, confirming AnnData-compliant per-modality storage
- After mdata.update(), joint .obs must contain modality-prefixed columns such as 'rna:n_genes' copied from modality-level annotations
- mu.read must dispatch automatically between read_h5mu and read_h5ad depending on the path
- mdata.copy('filename_copy.h5mu') must produce a valid backed copy when the object was read with backed=True

## Limitations

- Explicit support for genomic-coordinate based assays and assays with spatial coordinates is listed as future development, not current capability
- Explicitly modeling dependencies between feature sets across omics in MuData is planned for future development
- muon's mdata.copy backed-copy path applies specifically when the object was read with backed=True

## Evidence

- [results] In the MuData serialisation scheme, each individual omics layer is stored as an AnnData object using the existing AnnData serialisation format inside an HDF5 file: [evidence span withheld — anchor status: not_found]
- [methods] mu.read_snap ingestion example: "atac = ac.read_snap("pbmc3k.snap", matrix="bins", bin_size=10000)"
- [methods] read_10x_mtx directory files: "{func}`muon.read_10x_mtx` methods work in the same way for directories with files `matrix.mtx`, `features.tsv.gz`, and `barcodes.tsv.gz`."
- [discussion] HDF5 serialisation cross-language accessibility: "Serialisation to HDF5 makes MuData objects accessible to other programming languages, including R and Julia."
- [readme] muon is a multimodal omics Python framework: "`muon` is a multimodal omics Python framework."
- [methods] in-place filtering functions: "`muon` introduces functions for in-place filtering: {func}`muon.pp.filter_obs` and {func}`muon.pp.filter_var`"
