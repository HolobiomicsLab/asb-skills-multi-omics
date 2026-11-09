---
name: program-feature-score-pearson-correlation
description: Use when integrating cNMF-factorized program feature-score matrices (.h5ad)
  from cohorts with incomplete molecular-feature overlap, and you need anchored, highly
  correlated program pairs to build a cross-dataset program network.
when_to_use_negative:
- Datasets must first be factorized into programs (via cNMF); if input is still a
  raw expression or metadata table, run the mosaicMPI txt-to-h5ad and factorization
  steps first.
- Datasets that fully share all samples and features can be integrated by tools that
  operate on shared feature intersections; this skill targets the 'mosaic' case where
  cohorts do not all share the same samples or complete feature sets.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mosaicMPI
  role: Provides the anchoring workflow with dynamic thresholding; each step can be
    run as a subcommand within mosaicMPI
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: AnnData
  role: Input format for the factorized program feature-score matrices loaded before
    correlation
- name: pandas
  role: Handles the data matrices loaded for pairwise program comparison
- name: NumPy
  role: Compatible storage format for the pearson.df.npz correlation matrix output
- name: cNMF
  role: Consensus non-negative matrix factorization that produces the programs being
    correlated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:a460889fb5ca2637d1d579485c28502b30efdf3710b48de90bbdd01f294e9ccc
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# program-feature-score-pearson-correlation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes Pearson correlations between program feature-score matrices within and between CPTAC datasets (restricted to shared features), then derives an empirical null threshold from the mirrored negative correlation distribution to select anchor program pairs in the mosaicMPI integration framework.

## When to use

Use when integrating cNMF-factorized program feature-score matrices (.h5ad) from cohorts with incomplete molecular-feature overlap, and you need anchored, highly correlated program pairs to build a cross-dataset program network.

## When NOT to use

- Datasets must first be factorized into programs (via cNMF); if input is still a raw expression or metadata table, run the mosaicMPI txt-to-h5ad and factorization steps first.
- Datasets that fully share all samples and features can be integrated by tools that operate on shared feature intersections; this skill targets the 'mosaic' case where cohorts do not all share the same samples or complete feature sets.

## Inputs

- Factorized program feature-score matrices for each dataset as AnnData files (results/<dataset>/<dataset>.h5ad, e.g. cptac_snrna.h5ad)
- Datasets with incomplete overlap of molecular features (e.g. Microarray, RNA-Seq, Proteomics CPTAC modalities)

## Outputs

- pearson.df.npz — pearson correlation matrix of all programs within and between datasets, compatible with NumPy
- pairwise_corr_thresholds.txt — 95th-percentile dynamic thresholds per dataset and dataset pair
- anchor_edge_list.tsv — edge list of program pairs whose correlation exceeds the dataset-pair threshold

## How to apply

Load the factorized program feature-score matrices for each dataset (e.g. results/<dataset>/<dataset>.h5ad) using mosaicmpi/AnnData and pandas. For every within- and between-dataset program pair, restrict the comparison to the shared features between the two datasets and compute Pearson correlations, saving the result as pearson.df.npz in NumPy-compatible format. Build the empirical null distribution of correlations from the mirrored negative part of the correlation distribution and take its 95th percentile as the threshold per dataset and per dataset pair, which calibrates for systematic between-dataset differences and writes pairwise_corr_thresholds.txt. Retain program pairs whose correlation exceeds their dataset-pair threshold as anchor edges in anchor_edge_list.tsv; this dynamic thresholding is mosaicMPI's anchoring mechanism instead of a fixed correlation cutoff.

## Related tools

- **mosaicMPI** (Provides the anchoring workflow with dynamic thresholding; each step can be run as a subcommand within mosaicMPI) — https://github.com/MorrissyLab/mosaicMPI
- **AnnData** (Input format for the factorized program feature-score matrices loaded before correlation)
- **pandas** (Handles the data matrices loaded for pairwise program comparison)
- **NumPy** (Compatible storage format for the pearson.df.npz correlation matrix output)
- **cNMF** (Consensus non-negative matrix factorization that produces the programs being correlated)

## Evaluation signals

- pearson.df.npz loads as a NumPy-compatible symmetric correlation matrix covering all programs within and between the datasets.
- pairwise_corr_thresholds.txt contains one 95th-percentile threshold per dataset and per dataset pair, each derived from the mirrored negative correlation null distribution.
- Every edge in anchor_edge_list.tsv has a correlation strictly above its dataset-pair threshold; no below-threshold pairs are retained.
- For the three CPTAC modalities, the anchor edge list contains approximately 50,424 anchor pairs (~6.1% of all pairwise correlations).

## Limitations

- A potential concern with mosaic integration is loss of relevant information between modalities given that not all features are shared.
- Datasets have incomplete overlap of molecular features, so correlations must be computed only on shared features, and systematic between-dataset differences require the per-pair calibrated threshold.
- cNMF supports input data that is sparse (i.e. with zeros), but not with missing values, so missing-value inputs must be resolved before program factorization.

## Evidence

- [readme] High performance integration of datasets with mismatched features (eg. Microarray, RNA-Seq, Proteomics) or sparsity: [evidence span withheld — anchor status: external_doc]
