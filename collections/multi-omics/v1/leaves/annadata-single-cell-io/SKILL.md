---
name: annadata-single-cell-io
description: Use when raw expression matrices (e.g. txt) and metadata must be converted
  to AnnData .h5ad, or when mosaicMPI/cNMF results (program feature scores and usages)
  must be stored per dataset or reloaded from AnnData files.
when_to_use_negative:
- Input matrices contain missing (NaN) values — cNMF supports sparse data with zeros
  but not missing values, so the data must be preprocessed first
- Datasets are already stored as AnnData .h5ad objects — mosaicMPI imports them directly,
  so no txt-to-h5ad conversion is needed
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: mosaicMPI
  role: Provides the txt-to-h5ad subcommand for conversion and writes per-dataset
    .h5ad results containing cNMF programs and usages
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: AnnData
  role: Container format in which per-dataset expression data and cNMF program results
    are stored and exchanged (.h5ad)
- name: pandas
  role: Source format for data matrices that can be made into AnnData objects
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:388c176fd1b38b0fe2547039ec97ad260c1d831ef77d04472efca2e92b68786e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/MorrissyLab/mosaicMPI
  repo_url: https://github.com/MorrissyLab/mosaicMPI
  tools:
  - AnnData
  - cNMF
  - mosaicMPI
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# annadata-single-cell-io

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Single-cell and bulk expression data I/O in the mosaicMPI framework: converting raw txt expression matrices plus metadata into AnnData .h5ad objects, and persisting each dataset's input data together with cNMF program feature scores and usages at every retained rank as a per-dataset .h5ad file.

## When to use

Use when raw expression matrices (e.g. txt) and metadata must be converted to AnnData .h5ad, or when mosaicMPI/cNMF results (program feature scores and usages) must be stored per dataset or reloaded from AnnData files.

## When NOT to use

- Input matrices contain missing (NaN) values — cNMF supports sparse data with zeros but not missing values, so the data must be preprocessed first
- Datasets are already stored as AnnData .h5ad objects — mosaicMPI imports them directly, so no txt-to-h5ad conversion is needed

## Inputs

- Tab-delimited expression matrix (e.g. cptac_snRNA_subsampled.txt)
- Sample/cell metadata table (e.g. cptac_snRNA_subsampled.metadata.txt)
- pandas data matrices
- Existing AnnData files/objects

## Outputs

- AnnData .h5ad file per dataset (e.g. cptac_snrna.h5ad) containing the input data plus cNMF program feature scores and usages at each retained rank

## How to apply

Convert tab-delimited expression data and its metadata table into AnnData using the mosaicmpi txt-to-h5ad subcommand (e.g. producing cptac_snrna.h5ad from cptac_snRNA_subsampled.txt and cptac_snRNA_subsampled.metadata.txt). mosaicMPI accepts data as pandas data matrices or imported from AnnData files/objects, so existing .h5ad inputs can be loaded directly. After cNMF factorization across the k = 2-60 rank range, write each dataset's results — the input data plus cNMF program feature scores and usages at each retained rank — to a per-dataset .h5ad file such as results/cptac_snrna/cptac_snrna.h5ad. Ensure matrices are sparse-compatible (zeros allowed) before conversion, since cNMF supports sparse input data but not missing values.

## Related tools

- **mosaicMPI** (Provides the txt-to-h5ad subcommand for conversion and writes per-dataset .h5ad results containing cNMF programs and usages) — https://github.com/MorrissyLab/mosaicMPI
- **AnnData** (Container format in which per-dataset expression data and cNMF program results are stored and exchanged (.h5ad))
- **pandas** (Source format for data matrices that can be made into AnnData objects)

## Examples

```
mosaicmpi txt-to-h5ad --data cptac_snRNA_subsampled.txt --metadata cptac_snRNA_subsampled.metadata.txt -o cptac_snrna.h5ad
```

## Evaluation signals

- The output .h5ad file loads successfully as an AnnData object and contains the original expression data plus metadata
- Each per-dataset .h5ad (e.g. results/cptac_snrna/cptac_snrna.h5ad) contains cNMF program feature scores and usages for every retained rank in the k = 2-60 range
- mosaicmpi postprocess confirms that every factorization completed before results are written to the .h5ad files
- The expression matrix dimensions are consistent with the metadata table rows used in txt-to-h5ad

## Limitations

- cNMF input data must be sparse (with zeros) but not with missing values — NaNs are not supported
- Memory usage depends on the size and number of datasets being stored in AnnData objects
- No software version information is stated for the tools/methods in this section

## Evidence

- [methods] These can be made from from pandas data matrices or imported from AnnData files/objects.: "These can be made from from pandas data matrices or imported from AnnData files/objects."
- [methods] mosaicmpi txt-to-h5ad command: "mosaicmpi txt-to-h5ad --data cptac_snRNA_subsampled.txt --metadata cptac_snRNA_subsampled.metadata.txt -o cptac_snrna.h5ad"
- [methods] cNMF sparse input constraint: "cNMF  supports input data that is sparse (i.e. with zeros), but not with missing values."
- [readme] mosaicMPI package purpose: "mosaicMPI is a Python package for enabling mosaic integration of bulk, single-cell, and spatial expression data through program-level integration."
