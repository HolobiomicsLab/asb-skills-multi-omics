---
name: multi-omics-count-matrix-processing
description: Use when raw count matrices from different -omics modalities (bulk RNA,
  mass spectrometry proteomics, snRNA-Seq) must be converted to mosaicMPI's h5ad inputs
  and checked for missing, negative, or zero-variance features before overdispersion
  modeling.
when_to_use_negative:
- Input is already a validated, normalized feature table that has passed check-h5ad
  — proceed directly to select-hvf instead
- Data contains missing values that cannot be resolved per mosaicMPI's sparse-compatible
  rules (zeros are supported, missing values are not)
- Datasets that are not count-style expression matrices (e.g. already-aggregated program
  or correlation outputs such as pearson.df.npz)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: mosaicMPI
  role: Provides the CLI subcommands (txt-to-h5ad, check-h5ad) used to convert and
    validate count matrices before feature selection
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Loads raw count matrices and metadata into data frames that can be written
    to AnnData objects
- name: AnnData
  role: Container format imported/exported as .h5ad for per-dataset count matrices
- name: NumPy
  role: Numerical backend for handling the count matrices and correlation-compatible
    outputs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:ac4ba4d0e3a38faf6e84216369e0af5a3e984d9df3870be012f0f7a176432a09
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
  - NumPy
  - mosaicMPI
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.25
n_claims: 8
---

# multi-omics-count-matrix-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Load and validate heterogeneous CPTAC-3 GBM count matrices (bulk RNA-Seq STAR counts, PDC000204 proteomics, snRNA-Seq feature counts) into mosaicMPI-compatible AnnData/h5ad objects with modality-specific normalization and missing-value handling, so that downstream feature selection and program discovery operate on valid per-dataset inputs.

## When to use

Use when raw count matrices from different -omics modalities (bulk RNA, mass spectrometry proteomics, snRNA-Seq) must be converted to mosaicMPI's h5ad inputs and checked for missing, negative, or zero-variance features before overdispersion modeling.

## When NOT to use

- Input is already a validated, normalized feature table that has passed check-h5ad — proceed directly to select-hvf instead
- Data contains missing values that cannot be resolved per mosaicMPI's sparse-compatible rules (zeros are supported, missing values are not)
- Datasets that are not count-style expression matrices (e.g. already-aggregated program or correlation outputs such as pearson.df.npz)

## Inputs

- CPTAC-3 GBM bulk RNA-Seq STAR count matrix
- CPTAC-3 PDC000204 proteomics count matrix
- CPTAC-3 snRNA-Seq feature count matrix
- Per-dataset metadata tables (whitespace-delimited, as used by txt-to-h5ad)

## Outputs

- Per-dataset .h5ad AnnData files (e.g. cptac_snrna.h5ad)
- Normalized per-dataset expression matrices ready for mean-variance modeling
- Feature-validity report from check-h5ad (missing/negative/zero-variance features removed)

## How to apply

Load each CPTAC-3 GBM count matrix with pandas/AnnData, treating missing values according to mosaicMPI's sparse-compatible input rules — cNMF supports data that is sparse (i.e. with zeros) but not with missing values. Convert tabular data and metadata with the mosaicmpi txt-to-h5ad subcommand to produce per-dataset .h5ad files. Run mosaicmpi's check-h5ad validation to remove invalid features (missing values, negative values, zero variance) before any variance modeling. Produce a normalized matrix for each dataset so that modality-specific mean-variance dependence can be modeled in the subsequent overdispersion/HVF selection step. Rationale: distinct data types have heterogeneous statistical properties, so each modality requires bespoke preprocessing prior to integration.

## Related tools

- **mosaicMPI** (Provides the CLI subcommands (txt-to-h5ad, check-h5ad) used to convert and validate count matrices before feature selection) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Loads raw count matrices and metadata into data frames that can be written to AnnData objects)
- **AnnData** (Container format imported/exported as .h5ad for per-dataset count matrices)
- **NumPy** (Numerical backend for handling the count matrices and correlation-compatible outputs)

## Examples

```
mosaicmpi txt-to-h5ad --data cptac_snRNA_subsampled.txt --metadata cptac_snRNA_subsampled.metadata.txt -o cptac_snrna.h5ad
```

## Evaluation signals

- Each .h5ad output exists and is loadable as an AnnData object after txt-to-h5ad conversion
- check-h5ad reports no remaining features with missing values, negative values, or zero variance
- Downstream overdispersed_genes.csv per-dataset counts approximate the reported 8454 (bulk RNA), 4541 (protein), and 7072 (snRNA) features
- Dataset-specific normalization applied per modality rather than a single shared transform across modalities
- mosaicmpi --help confirms the expected subcommands are available in the installed environment

## Limitations

- cNMF/mosaicMPI inputs support zeros but not missing values, so matrices with true NA entries must be handled before factorization
- Potential loss of relevant information between modalities in mosaic integration because not all features are shared
- No software version information is stated in the article for the tools/methods discussed

## Evidence
