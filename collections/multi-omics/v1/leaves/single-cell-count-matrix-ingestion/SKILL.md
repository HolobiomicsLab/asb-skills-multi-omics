---
name: single-cell-count-matrix-ingestion
description: Use when the input is raw single-cell assay output such as a Cell Ranger
  filtered_feature_bc_matrix.h5 file, a matrix.mtx/features.tsv.gz/barcodes.tsv.gz
  directory, or a SnapATAC .snap file, and you need AnnData objects per modality for
  multimodal (e.g. RNA+ATAC) analysis.
when_to_use_negative:
- The data is already an AnnData (.h5ad) or MuData (.h5mu) object — reload with mu.read
  instead of re-ingesting raw matrices.
- The input is a feature/peak table or already-normalised matrix rather than raw per-cell
  count output in 10x or snap format.
- The data lives outside muon's supported ingestion formats (e.g. needs custom parsing)
  with no 10x or snap equivalent.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: MUON
  role: Provides mu.read_10x_h5, mu.read_10x_mtx, and the ATAC module's ac.read_snap
    to ingest count matrices into AnnData per modality
  repo: https://github.com/scverse/muon
- name: SCANPY
  role: Underlying AnnData-based single-cell framework compatible with the ingested
    count matrices
- name: NumPy
  role: Array backend backing the count matrices in the ingested AnnData objects
- name: pandas
  role: DataFrame backend for the .obs/.var annotations attached during ingestion
- name: SnapATAC
  role: Origin format of .snap single-nucleus ATAC files read via ac.read_snap with
    matrix='bins' and bin_size=10000
  repo: https://github.com/r3fang/SnapATAC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:985005a5994c25dfeab6b3c6f530e0b4bd15eebba062359b962d7a727d440ea7
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# Single-cell count matrix ingestion

## Summary

Ingest raw single-cell count matrices from standard 10x Genomics and SnapATAC formats into AnnData objects per modality using muon's readers, as the first step of assembling a MuData multimodal container. This ensures each omics layer starts as a standard AnnData that can later be serialised to HDF5/.h5mu.

## When to use

Use when the input is raw single-cell assay output such as a Cell Ranger filtered_feature_bc_matrix.h5 file, a matrix.mtx/features.tsv.gz/barcodes.tsv.gz directory, or a SnapATAC .snap file, and you need AnnData objects per modality for multimodal (e.g. RNA+ATAC) analysis.

## When NOT to use

- The data is already an AnnData (.h5ad) or MuData (.h5mu) object — reload with mu.read instead of re-ingesting raw matrices.
- The input is a feature/peak table or already-normalised matrix rather than raw per-cell count output in 10x or snap format.
- The data lives outside muon's supported ingestion formats (e.g. needs custom parsing) with no 10x or snap equivalent.

## Inputs

- Cell Ranger filtered_feature_bc_matrix.h5 file (10x Genomics HDF5 count matrix)
- Directory with matrix.mtx, features.tsv.gz, and barcodes.tsv.gz files (10x mtx format)
- SnapATAC .snap file with single-nucleus accessibility profiles

## Outputs

- AnnData object per modality (e.g. adata_rna, adata_atac) with count matrices in .X
- MuData multimodal container (modalities under .mod, keyed by name) ready for mdata.write() to .h5mu

## How to apply

Pick the reader matching the input format: mu.read_10x_h5 for a Cell Ranger filtered_feature_bc_matrix.h5 file, or mu.read_10x_mtx for directories containing matrix.mtx, features.tsv.gz, and barcodes.tsv.gz; these methods work in the same way for the mtx directory layout. For single-nucleus ATAC-seq snap files, use the ATAC module reader with matrix='bins' and bin_size=10000 (e.g. ac.read_snap("pbmc3k.snap", matrix="bins", bin_size=10000)). Each reader returns a standard AnnData object, one per modality, which can then be assembled into a MuData container via mu.MuData({'rna': adata_rna, 'atac': adata_atac}). Because each modality is stored as AnnData in the MuData serialisation scheme, single omics remain accessible directly with existing toolchains. After ingestion, call mdata.update() to sync joint .obs/.var annotations before serialising with mdata.write('mudata.h5mu').

## Related tools

- **MUON** (Provides mu.read_10x_h5, mu.read_10x_mtx, and the ATAC module's ac.read_snap to ingest count matrices into AnnData per modality) — https://github.com/scverse/muon
- **SCANPY** (Underlying AnnData-based single-cell framework compatible with the ingested count matrices)
- **NumPy** (Array backend backing the count matrices in the ingested AnnData objects)
- **pandas** (DataFrame backend for the .obs/.var annotations attached during ingestion)
- **SnapATAC** (Origin format of .snap single-nucleus ATAC files read via ac.read_snap with matrix='bins' and bin_size=10000) — https://github.com/r3fang/SnapATAC

## Examples

```
import muon as mu; adata_rna = mu.read_10x_h5('filtered_feature_bc_matrix.h5'); atac = mu.atac.ac.read_snap('pbmc3k.snap', matrix='bins', bin_size=10000)
```

## Evaluation signals

- Each reader call returns an AnnData object (not a raw dict or sparse file handle) with cells in .obs and features in .var.
- mu.MuData({'rna': adata_rna, 'atac': adata_atac}) builds a container whose .mod attribute exposes both modalities keyed by name.
- After mdata.write('mudata.h5mu'), h5ls exposes a top-level mod Group with atac and rna subgroups alongside obs/obsm/var/varm groups.
- mu.read('mudata.h5mu/rna') returns an AnnData object without error, confirming AnnData-compliant per-modality storage.
- mu.read dispatches automatically between read_h5mu and read_h5ad on the written file.

## Limitations

- Reading a snap file requires binning parameters (matrix='bins', bin_size=10000); other bin sizes or matrices are not covered by the documented example.
- The mtx reader path expects exactly the files matrix.mtx, features.tsv.gz, and barcodes.tsv.gz in the directory.
- Emerging multi-omics assays and genomic-coordinate or spatially resolved assays are not yet supported ingestion targets in MUON and are listed as future work.

## Evidence

- [methods] with {func}`muon.read_10x_h5`: "with {func}`muon.read_10x_h5`"
- [methods] read_10x_mtx methods work in the same way for directories with files matrix.mtx, features.tsv.gz, and barcodes.tsv.gz.: "{func}`muon.read_10x_mtx` methods work in the same way for directories with files `matrix.mtx`, `features.tsv.gz`, and `barcodes.tsv.gz`."
- [methods] atac = ac.read_snap("pbmc3k.snap", matrix="bins", bin_size=10000): "atac = ac.read_snap("pbmc3k.snap", matrix="bins", bin_size=10000)"
- [abstract] we present a data standard and an analysis framework for multi-omics, MUON, designed to organise, analyse, visualise, and exchange multimodal data: "we present a data standard and an analysis framework for multi-omics, MUON, designed to organise, analyse, visualise, and exchange multimodal data"
- [methods] [Snap files](https://github.com/r3fang/SnapATAC/wiki/FAQs#whatissnap) describe single-nucleus accessibility profiles.: "[Snap files](https://github.com/r3fang/SnapATAC/wiki/FAQs#whatissnap) describe single-nucleus accessibility profiles."
