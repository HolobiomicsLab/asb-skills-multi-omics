---
name: cross-cohort-program-network-integration
description: Use when two or more expression datasets (bulk/single-cell/spatial, possibly
  with mismatched features and no shared sample IDs) must be integrated at the program
  level and annotations such as transcriptional subtypes propagated from a reference
  cohort to a new cohort.
when_to_use_negative:
- Datasets already share all samples and features and can be jointly analyzed without
  mosaic integration
- The goal is single-dataset exploration only — mosaicMPI program discovery within
  one dataset does not require cross-cohort integration
- No annotated reference cohort or metadata layer exists to propagate onto the program
  network
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: mosaicMPI
  role: Command-line framework that builds the cross-cohort program network, detects
    communities (max_k_filter/max_k_median_corr), and performs label transfer via
    'mosaicmpi integrate' and 'mosaicmpi transfer-labels'
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: cNMF
  role: Consensus non-negative matrix factorization that discovers the low- to high-resolution
    programs within each dataset that become network nodes; factorization on over-dispersed
    features with NNLS re-fit across all features
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: AnnData
  role: h5ad container format holding expression, metadata, cNMF programs and usages
    passed to the integration workflow
- name: pandas
  role: Source of data matrices that can be converted to AnnData objects for mosaicMPI
    input
- name: NumPy
  role: Reads pearson.df.npz, the Pearson correlation matrix of all programs within
    and between datasets underlying the network
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:8ef24535d9c8074ab9f364937477187f516d149dae99f1dccb7d42a50b6809ac
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# cross-cohort-program-network-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrates independently factorized datasets (e.g. Mayo-PDX bulk RNA-Seq and CPTAC GBM) into a cross-cohort program network using mosaicMPI, where program communities anchor metadata/subtype label transfer between cohorts without re-analysis or feature intersection.

## When to use

Use when two or more expression datasets (bulk/single-cell/spatial, possibly with mismatched features and no shared sample IDs) must be integrated at the program level and annotations such as transcriptional subtypes propagated from a reference cohort to a new cohort.

## When NOT to use

- Datasets already share all samples and features and can be jointly analyzed without mosaic integration
- The goal is single-dataset exploration only — mosaicMPI program discovery within one dataset does not require cross-cohort integration
- No annotated reference cohort or metadata layer exists to propagate onto the program network

## Inputs

- Bulk RNA-Seq expression matrix and sample metadata (e.g. Mayo-PDX GBM, n=66) as txt files for 'mosaicmpi txt-to-h5ad'
- AnnData h5ad files per dataset (e.g. cptac_snrna.h5ad, Mayo-PDX h5ad) with cNMF consensus programs and usages
- Annotated reference cohort metadata layer (e.g. CPTAC transcriptional subtype labels)
- TOML configuration file listing the datasets to integrate

## Outputs

- network_integration.pkl.gz — the integrated cross-cohort program network object
- Program communities grouping similar programs across datasets and resolutions
- Transferred subtype labels for destination-cohort samples (Mayo-PDX)
- Label-transfer concordance statistics (e.g. ~72% overall concordance vs. Mayo-based subtypes)

## How to apply

First factorize each dataset independently with cNMF over a rank range (e.g. k=2 to 60 via 'mosaicmpi initialize-cnmf --k_range 2 60 1', then factorize and postprocess) after selecting highly-variable features ('mosaicmpi select-hvf default', which models mean-variance dependence on normalized data with F-distribution p-values, alpha 0.05). Write a TOML configuration listing both datasets ('mosaicmpi create-config'), then run 'mosaicmpi integrate -c config.toml -o output_directory' to build the Pearson-correlated program network and detect communities, using max_k_filter based on max_k_median_corr to drop unreliable ranks. Communities containing programs from both datasets act as anchors, so clinical/molecular annotations (e.g. CPTAC transcriptional subtypes) can be statistically propagated across the network. Transfer labels with 'mosaicmpi transfer-labels -n network_integration.pkl.gz' using the reference dataset (CPTAC) as source and the new dataset (Mayo-PDX) as destination, then quantify concordance between transferred and native labels — the GBM Mayo-PDX benchmark achieved ~72% overall concordance (Figure 6G).

## Related tools

- **mosaicMPI** (Command-line framework that builds the cross-cohort program network, detects communities (max_k_filter/max_k_median_corr), and performs label transfer via 'mosaicmpi integrate' and 'mosaicmpi transfer-labels') — https://github.com/MorrissyLab/mosaicMPI
- **cNMF** (Consensus non-negative matrix factorization that discovers the low- to high-resolution programs within each dataset that become network nodes; factorization on over-dispersed features with NNLS re-fit across all features) — https://github.com/MorrissyLab/mosaicMPI
- **AnnData** (h5ad container format holding expression, metadata, cNMF programs and usages passed to the integration workflow)
- **pandas** (Source of data matrices that can be converted to AnnData objects for mosaicMPI input)
- **NumPy** (Reads pearson.df.npz, the Pearson correlation matrix of all programs within and between datasets underlying the network)

## Examples

```
mosaicmpi integrate -c config.toml -o output_directory && mosaicmpi transfer-labels -n network_integration.pkl.gz
```

## Evaluation signals

- Overall label-transfer concordance between CPTAC-derived and Mayo-based transcriptional subtypes reaches approximately 72% (Figure 6G benchmark)
- network_integration.pkl.gz is produced in the output directory and contains program Pearson correlations and community assignments
- Communities are formed only at ranks passing the max_k_filter based on max_k_median_corr
- Every input dataset has independently run cNMF consensus programs/usages stored in its h5ad before integration
- The TOML config lists both the source (CPTAC) and destination (Mayo-PDX) datasets with the subtype layer designated for transfer

## Limitations

- Mosaic integration may lose relevant information between modalities because not all features are shared across datasets
- Datasets have incomplete overlap of molecular features and no shared sample IDs, so integration occurs at the program level rather than feature or sample level
- No software version numbers are stated in the article; check 'mosaicmpi --help' for available subcommands in your installed version

## Evidence

- [readme] Mosaic integration does not require subsetting features/genes to intersection or highly-variable subset: [evidence span withheld — anchor status: external_doc]
