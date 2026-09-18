---
name: mudata-container-management
description: Use when a task involves paired multi-omics single-cell data (e.g. 10x
  Multiome filtered_feature_bc_matrix.h5 with 'rna' and 'atac' modalities) that must
  be loaded, filtered, aligned across modalities, or saved as a single container for
  joint analysis.
when_to_use_negative:
- Input is a single-omics dataset (one AnnData object) with no paired modalities —
  plain scanpy workflows suffice
- Data are already stored in an interoperable multi-modal container and only single-modality
  operations are needed
- The assay requires genomic-coordinate or spatial-coordinate handling, which MUON
  lists as future development rather than current capability
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3170
tools:
- name: MUON
  role: Core framework providing MuData container I/O (read_10x_h5, read_10x_mtx),
    in-place filtering (filter_obs, filter_var, intersect_obs), and .h5mu serialisation
  repo: https://github.com/scverse/muon
- name: SCANPY
  role: Performs single-modality preprocessing (normalise_per_cell, log1p) on the
    RNA modality within the MuData container
- name: MOFA+
  role: Jointly models modalities stored in the MuData container via mu.tl.mofa, writing
    factors to mdata.obsm['X_mofa']
- name: SnapATAC
  role: Alternative single-cell ATAC-seq analysis pipeline whose snap files can be
    read via muon.atac.read_snap
  repo: https://github.com/r3fang/SnapATAC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:58ba9f27777f204870f4a1d03e4b74297f13d0c2f3e3e1c840832dec615dafd5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/muon
  tools:
  - MOFA+
  - MUON
  - SCANPY
  - SnapATAC
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: thin
yes_rate: 0.0
n_claims: 0
---

# mudata-container-management

## Summary

Create, manipulate, and serialise MuData multimodal omics containers using the MUON framework, which organises paired modalities (e.g. scRNA-seq and scATAC-seq) in an interoperable HDF5-backed structure for downstream integration such as MOFA factor analysis.

## When to use

Use when a task involves paired multi-omics single-cell data (e.g. 10x Multiome filtered_feature_bc_matrix.h5 with 'rna' and 'atac' modalities) that must be loaded, filtered, aligned across modalities, or saved as a single container for joint analysis.

## When NOT to use

- Input is a single-omics dataset (one AnnData object) with no paired modalities — plain scanpy workflows suffice
- Data are already stored in an interoperable multi-modal container and only single-modality operations are needed
- The assay requires genomic-coordinate or spatial-coordinate handling, which MUON lists as future development rather than current capability

## Inputs

- 10x Genomics multiome HDF5 feature-barcode matrix (filtered_feature_bc_matrix.h5)
- 10x MTX directory with matrix.mtx, features.tsv.gz, and barcodes.tsv.gz
- Existing MuData/AnnData objects requiring cross-modality alignment or filtering

## Outputs

- MuData object with 'rna' and 'atac' modalities and obsm embeddings (e.g. X_mofa)
- Serialised .h5mu HDF5 file (e.g. mofa_pbmc_multiome.h5mu)

## How to apply

Read 10x Genomics multiome data with mu.read_10x_h5 (or mu.read_10x_mtx for matrix.mtx/features.tsv.gz/barcodes.tsv.gz directories) to build a MuData object holding 'rna' and 'atac' modalities. Keep observations aligned across modalities — use mu.pp.intersect_obs to retain cells present in all modalities, and mu.pp.filter_obs / mu.pp.filter_var for in-place cell and feature filtering. Apply modality-specific preprocessing inside the container (e.g. scanpy log-normalisation with counts_per_cell_after=1e4 then sc.pp.log1p for RNA; ac.pp.tfidf or ac.pp.binarize for ATAC) before multi-omics integration. Serialise the container with mdata.write to .h5mu (HDF5), which makes MuData objects accessible from other languages such as R and Julia. MUON's modular design means existing single-omics tools (scanpy for RNA, ATAC module utilities) operate on individual modalities within the shared container.

## Related tools

- **MUON** (Core framework providing MuData container I/O (read_10x_h5, read_10x_mtx), in-place filtering (filter_obs, filter_var, intersect_obs), and .h5mu serialisation) — https://github.com/scverse/muon
- **SCANPY** (Performs single-modality preprocessing (normalise_per_cell, log1p) on the RNA modality within the MuData container)
- **MOFA+** (Jointly models modalities stored in the MuData container via mu.tl.mofa, writing factors to mdata.obsm['X_mofa'])
- **SnapATAC** (Alternative single-cell ATAC-seq analysis pipeline whose snap files can be read via muon.atac.read_snap) — https://github.com/r3fang/SnapATAC

## Examples

```
import muon as mu; mdata = mu.read_10x_h5('filtered_feature_bc_matrix.h5'); mu.pp.intersect_obs(mdata); mdata.write('mofa_pbmc_multiome.h5mu')
```

## Evaluation signals

- mdata.obsm['X_mofa'] exists after mu.tl.mofa (the workflow checks that "'X_mofa' in mdata.obsm" evaluates to True)
- MuData object contains both 'rna' and 'atac' modalities after mu.read_10x_h5 on a 10x multiome HDF5 file
- Cell (obs) sets are consistent across modalities after mu.pp.intersect_obs — no cells lost from one modality only
- mdata.write('...h5mu') produces a valid HDF5 file that can be re-read, reflecting HDF5 serialisation accessible from R and Julia
- RNA modality shows normalised, log1p-transformed values after sc.pp.normalise_per_cell(counts_per_cell_after=1e4) followed by sc.pp.log1p

## Limitations

- Support for emerging multi-omics assays is future work: 'Looking ahead, MUON will be a robust platform to build upon and support future developments.'
- Explicit support for genomic-coordinate based assays and assays with spatial coordinates is listed as future development
- No open exchange format accessible from different programming languages existed prior to MuData's HDF5 serialisation, so .h5mu interoperability depends on HDF5 bindings in the target language

## Evidence

- [abstract] MUON is a data standard and analysis framework for multi-omics designed to organise, analyse, visualise, and exchange multimodal data.: "we present a data standard and an analysis framework for multi-omics, MUON, designed to organise, analyse, visualise, and exchange multimodal data"
- [discussion] HDF5 serialisation enables cross-language access to MuData objects.: "Serialisation to HDF5 makes MuData objects accessible to other programming languages, including R and Julia."
- [methods] In-place filtering and cross-modality observation intersection functions.: "`muon` introduces functions for in-place filtering: {func}`muon.pp.filter_obs` and {func}`muon.pp.filter_var`"
- [methods] Reading 10x multiome HDF5 files into MuData containers.: "with {func}`muon.read_10x_h5`"
- [readme] muon is a multimodal omics Python framework (README).: "`muon` is a multimodal omics Python framework."
