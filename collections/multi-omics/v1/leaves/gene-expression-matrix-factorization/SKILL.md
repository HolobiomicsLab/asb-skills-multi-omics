---
name: gene-expression-matrix-factorization
description: Use when bulk, single-cell, or spatial RNA-Seq expression matrices need
  unsupervised decomposition into interpretable programs, or when cohorts with mismatched
  features/samples must be integrated without subsetting to a shared gene set.
when_to_use_negative:
- Input data contain missing values — cNMF supports sparse (zero) data but not missing
  values
- Datasets already share all samples and features and can be analyzed in a single
  joint factorization without mosaic integration
- The goal is feature-level (gene-level) integration rather than program-level integration
  across cohorts/modalities
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: mosaicMPI
  role: CLI/Python framework orchestrating HVF selection, cNMF factorization, program-network
    integration, and metadata/label transfer
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: cNMF
  role: Consensus non-negative matrix factorization engine used to discover programs
    over a k range within each dataset
- name: AnnData
  role: H5ad object format holding expression matrices, metadata, and program usages
- name: pandas
  role: Source of data matrices that can be imported into AnnData objects
- name: NumPy
  role: Format for the output program correlation matrix (pearson.df.npz)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:48d863539368abb37e57affa138dae3620b808fdfe54d369d3f62348ae32050a
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
  - cNMF
  - mosaicMPI
  - pandas
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# gene-expression-matrix-factorization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Discovers low- to high-resolution molecular programs from gene expression matrices via consensus non-negative matrix factorization (cNMF) across a rank range, then integrates programs across datasets into a network for downstream label transfer.

## When to use

Use when bulk, single-cell, or spatial RNA-Seq expression matrices need unsupervised decomposition into interpretable programs, or when cohorts with mismatched features/samples must be integrated without subsetting to a shared gene set.

## When NOT to use

- Input data contain missing values — cNMF supports sparse (zero) data but not missing values
- Datasets already share all samples and features and can be analyzed in a single joint factorization without mosaic integration
- The goal is feature-level (gene-level) integration rather than program-level integration across cohorts/modalities

## Inputs

- Gene expression matrix (txt) with sample/cell metadata for conversion to AnnData .h5ad
- AnnData h5ad objects per dataset (bulk RNA-Seq, snRNA-Seq, or spatial)
- TOML configuration file listing datasets to integrate
- Optional custom HVF list (whitespace-delimited file) or annotation layers (e.g. transcriptional subtypes) for label transfer

## Outputs

- Consensus cNMF programs and usages stored in the dataset h5ad
- Pearson correlation matrix of all programs within and between datasets (pearson.df.npz)
- Cross-cohort program network with communities and network_integration.pkl.gz
- Transferred metadata/subtype labels on the destination dataset (e.g. CPTAC-to-Mayo label transfer)

## How to apply

Import expression data and metadata into an AnnData h5ad (e.g. with 'mosaicmpi txt-to-h5ad'), then select highly-variable features with 'mosaicmpi select-hvf default', which models mean-variance dependence on normalized data with F-distribution p-values at --alpha 0.05, optionally stratified by patient. Run cNMF factorization over a rank range (e.g. 'mosaicmpi initialize-cnmf --k_range 2 60 1', then 'factorize' and 'postprocess') to obtain consensus programs and usages; cNMF factorizes on the over-dispersed feature subset but non-negative least squares re-fits programs across all features. Build a TOML config listing each dataset and run 'mosaicmpi integrate' to construct the cross-cohort program network, where communities are formed with a max_k_filter based on max_k_median_corr. Because deconvolution is per-dataset, integration is incremental and mosaic: datasets need not share samples, IDs, or the full feature set.

## Related tools

- **mosaicMPI** (CLI/Python framework orchestrating HVF selection, cNMF factorization, program-network integration, and metadata/label transfer) — https://github.com/MorrissyLab/mosaicMPI
- **cNMF** (Consensus non-negative matrix factorization engine used to discover programs over a k range within each dataset)
- **AnnData** (H5ad object format holding expression matrices, metadata, and program usages)
- **pandas** (Source of data matrices that can be imported into AnnData objects)
- **NumPy** (Format for the output program correlation matrix (pearson.df.npz))

## Examples

```
mosaicmpi initialize-cnmf --k_range 2 60 1 -o mayo.h5ad; mosaicmpi factorize; mosaicmpi postprocess
```

## Evaluation signals

- Rank selection: each k passes the max_k_filter based on max_k_median_corr; ranks failing this filter should be excluded from the integrated network
- HVF selection uses F-distribution p-values on modeled mean-variance dependence at --alpha 0.05; inspect the selected-feature set for expected variance structure
- Cross-dataset program correlation matrix (pearson.df.npz) should show strong within-community Pearson correlations spanning datasets
- Downstream sanity check: label transfer (e.g. CPTAC-to-Mayo-PDX subtype labels) should reach roughly the reported ~72% concordance
- Each mosaicmpi subcommand runs standalone; verify with 'mosaicmpi --help' that every workflow step completed and outputs (h5ad, pkl.gz) are present

## Limitations

- A potential concern with mosaic integration is loss of relevant information between modalities given that not all features are shared
- cNMF does not support missing values in the input data (sparse zeros are fine)
- Memory usage depends on the size and number of datasets; factorization over a wide k range may require job submission scripts (e.g. slurm_factorize.sh, slurm_postprocess.sh)

## Evidence
