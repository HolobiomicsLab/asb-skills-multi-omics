---
name: hvfeature-selection-overdispersion
description: Use when preparing a bulk or single-cell RNA-Seq expression matrix (e.g.
  an AnnData h5ad) for cNMF factorization or downstream integration in mosaicMPI,
  and over-dispersed genes must be identified to restrict factorization to informative
  features.
when_to_use_negative:
- When a curated custom feature list already exists — use 'mosaicmpi select-hvf custom'
  instead of statistical HVF selection
- When data contain missing (not just sparse zero) values — cNMF/mosaicMPI supports
  sparse data with zeros but not missing values
- When the analysis goal requires using all features directly for integration — mosaicMPI
  mosaic integration does not require subsetting genes to an HVF or intersection subset
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: mosaicMPI
  role: Provides the 'select-hvf default/custom/cnmf/stdeconvolve' subcommands that
    model mean-variance dependence on normalized data to identify highly-variable
    features
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: cNMF
  role: 'Downstream consumer: consensus NMF factorization is performed using the subset
    of over-dispersed features, then NNLS re-fits programs across all features'
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: AnnData
  role: File/object format (h5ad) holding the normalized expression matrix into which
    HVF selections are written
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:62ae53b6fe10afd323efcfd10e5ae1e433d959729d4cd784c98fd99bc7870562
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# hvfeature-selection-overdispersion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects highly-variable features (HVFs) for a dataset by modeling mean-variance dependence on normalized expression data and flagging over-dispersed features, as implemented in mosaicMPI's 'select-hvf default' and used prior to cNMF factorization.

## When to use

Use when preparing a bulk or single-cell RNA-Seq expression matrix (e.g. an AnnData h5ad) for cNMF factorization or downstream integration in mosaicMPI, and over-dispersed genes must be identified to restrict factorization to informative features.

## When NOT to use

- When a curated custom feature list already exists — use 'mosaicmpi select-hvf custom' instead of statistical HVF selection
- When data contain missing (not just sparse zero) values — cNMF/mosaicMPI supports sparse data with zeros but not missing values
- When the analysis goal requires using all features directly for integration — mosaicMPI mosaic integration does not require subsetting genes to an HVF or intersection subset

## Inputs

- AnnData h5ad file of normalized expression data (e.g. produced by mosaicmpi txt-to-h5ad)
- Optional custom feature list (whitespace-delimited file) for select-hvf custom
- Metadata table stratifying samples by patient/batch where appropriate

## Outputs

- Highly-variable feature annotations stored in the dataset h5ad (used by mosaicmpi initialize-cnmf / factorize)
- Set of over-dispersed features passing the F-distribution p-value threshold (alpha 0.05)

## How to apply

Run 'mosaicmpi select-hvf default' on the normalized expression data, which models the mean-variance dependence and computes F-distribution p-values to identify features whose variance exceeds the mean-variance trend (use --alpha 0.05 as the significance cutoff). Stratify HVF selection by patient where appropriate, e.g. for datasets with multiple patients or batches. Rationale: cNMF factorization is performed on the subset of over-dispersed features, after which non-negative least squares re-fits the identified programs across all features, so restricting factorization to HVFs improves signal detection while program discovery retains full-feature coverage downstream. Alternatives include 'select-hvf custom' (whitespace-delimited feature list), 'select-hvf cnmf', and 'select-hvf stdeconvolve' when automatic selection is unsuitable.

## Related tools

- **mosaicMPI** (Provides the 'select-hvf default/custom/cnmf/stdeconvolve' subcommands that model mean-variance dependence on normalized data to identify highly-variable features) — https://github.com/MorrissyLab/mosaicMPI
- **cNMF** (Downstream consumer: consensus NMF factorization is performed using the subset of over-dispersed features, then NNLS re-fits programs across all features) — https://github.com/MorrissyLab/mosaicMPI
- **AnnData** (File/object format (h5ad) holding the normalized expression matrix into which HVF selections are written)

## Examples

```
mosaicmpi select-hvf default -n cptac_snrna.h5ad --alpha 0.05
```

## Evaluation signals

- Selected HVFs show variance significantly above the fitted mean-variance trend at F-distribution p < 0.05 (--alpha 0.05)
- The h5ad object's HVF annotation is populated and 'mosaicmpi initialize-cnmf' proceeds without error using the selected features
- Stratified selection (by patient) yields feature sets that are not dominated by a single patient/batch when stratification is enabled
- cNMF postprocessing yields consensus programs whose usage spectra support downstream community detection (max_k_filter / max_k_median_corr checks pass)

## Limitations

- cNMF supports sparse input (with zeros) but not missing values — missing entries must be imputed or removed before HVF selection and factorization
- A potential concern with mosaic integration is loss of relevant information between modalities given that not all features are shared; HVF subsetting may further reduce cross-modality overlap

## Evidence

- [methods] select-hvf custom: [evidence span withheld — anchor status: not_found]
