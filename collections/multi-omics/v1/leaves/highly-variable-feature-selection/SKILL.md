---
name: highly-variable-feature-selection
description: Use when preparing normalized count matrices (e.g. bulk RNA-Seq STAR
  counts, CPTAC proteomics, snRNA-Seq feature counts) for factorization and you need
  a modality-agnostic overdispersed feature subset instead of cNMF's scRNA-Seq-calibrated
  Poisson model.
when_to_use_negative:
- Input data still contains missing values or negative/zero-variance features that
  have not been cleaned — mosaicmpi check-h5ad validation must pass before overdispersion
  modeling
- You already have a curated feature list — use mosaicmpi select-hvf custom with a
  whitespace-delimited file instead of re-fitting the mean-variance curve
- The dataset is a pure scRNA-Seq count matrix where cNMF's own Poisson-calibrated
  overdispersion model is the intended method (select-hvf cnmf)
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3173
tools:
- name: mosaicMPI
  role: Provides the select-hvf subcommand (stdeconvolve/cnmf/custom methods) that
    fits the mean-variance curve, computes OD scores, and writes selected-feature
    outputs
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Loads the per-dataset count matrices into data frames used to build AnnData
    objects for variance modeling
- name: AnnData
  role: Container format for the normalized matrices and metadata passed into mosaicmpi
    select-hvf
- name: NumPy
  role: Numerical backend for residual variance and OD score computation on the normalized
    matrices
- name: cNMF
  role: Alternative scRNA-Seq-calibrated Poisson feature-selection method available
    as select-hvf cnmf; the OD-score approach generalizes beyond it
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:e8dc402a1248101fd2abd231fe10355feae0ff56dce30de39f225ca472ab6a98
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 9
---

# highly-variable-feature-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects overdispersed/highly-variable features per dataset by fitting a smooth mean-variance curve to normalized data (STdeconvolve method) and ranking features by an overdispersion (OD) score computed from residuals, generalizing cNMF's Poisson-calibrated feature selection to modalities such as mass spectrometry proteomics.

## When to use

Use when preparing normalized count matrices (e.g. bulk RNA-Seq STAR counts, CPTAC proteomics, snRNA-Seq feature counts) for factorization and you need a modality-agnostic overdispersed feature subset instead of cNMF's scRNA-Seq-calibrated Poisson model.

## When NOT to use

- Input data still contains missing values or negative/zero-variance features that have not been cleaned — mosaicmpi check-h5ad validation must pass before overdispersion modeling
- You already have a curated feature list — use mosaicmpi select-hvf custom with a whitespace-delimited file instead of re-fitting the mean-variance curve
- The dataset is a pure scRNA-Seq count matrix where cNMF's own Poisson-calibrated overdispersion model is the intended method (select-hvf cnmf)

## Inputs

- Normalized count matrix per dataset (bulk RNA-Seq STAR counts, PDC000204 proteomics, snRNA-Seq feature counts) as pandas/AnnData (.h5ad) objects
- Optional whitespace-delimited file of custom features for select-hvf custom
- Sample/cell metadata (e.g. patient) for stratified feature selection

## Outputs

- overdispersed_genes.csv — per-dataset list of selected features (expected ~8454 bulk RNA, 4541 protein, 7072 snRNA genes for the GBM datasets)
- feature_mean_var.pdf — mean-variance relationship with fitted model and selected features highlighted in red
- feature_odscore.pdf — overdispersion score histogram
- h5ad with selected highly-variable features annotated for downstream cNMF factorization

## How to apply

Load each count matrix (e.g. CPTAC-3 GBM bulk RNA-Seq STAR counts, PDC000204 proteomics, snRNA-Seq feature counts) with pandas/AnnData and produce a normalized matrix for variance modeling, handling missing values per mosaicMPI's sparse-compatible input rules since cNMF supports zeros but not missing values. Remove invalid features (missing values, negative values, zero variance), then fit a smooth curve to the mean-variance dependence of the normalized data using the STdeconvolve method (Miller et al., Nat. Comm. 2022) as reimplemented in mosaicMPI's select-hvf. Compute the overdispersion (OD) score from model residuals and retain features with OD score > 1, optionally stratifying by metadata (e.g. patient) and taking the union via select-hvf --stratify_by. Diagnostics are the mean-variance plot with selected features in red and the OD score histogram, and the selected genes are written per dataset for downstream cNMF-style factorization.

## Related tools

- **mosaicMPI** (Provides the select-hvf subcommand (stdeconvolve/cnmf/custom methods) that fits the mean-variance curve, computes OD scores, and writes selected-feature outputs) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Loads the per-dataset count matrices into data frames used to build AnnData objects for variance modeling)
- **AnnData** (Container format for the normalized matrices and metadata passed into mosaicmpi select-hvf)
- **NumPy** (Numerical backend for residual variance and OD score computation on the normalized matrices)
- **cNMF** (Alternative scRNA-Seq-calibrated Poisson feature-selection method available as select-hvf cnmf; the OD-score approach generalizes beyond it)

## Examples

```
mosaicmpi select-hvf stdeconvolve -o hvf -n cptac_snrna.h5ad
```

## Evaluation signals

- overdispersed_genes.csv exists, is parseable, and per-dataset gene counts approximate the reported 8454 (bulk RNA-Seq), 4541 (protein), and 7072 (snRNA) genes
- feature_mean_var.pdf shows the fitted mean-variance curve with selected features (red) concentrated above the curve
- feature_odscore.pdf histogram confirms a plausible distribution of OD scores with the OD > 1 threshold separating the selected features
- No invalid features (missing values, negatives, zero variance) remain — mosaicmpi check-h5ad passes before modeling
- Selected features reproduce when select-hvf is re-run, or match a user-supplied list when select-hvf custom is used

## Limitations

- Mosaic integration can lose relevant information between modalities because not all features are shared across datasets
- cNMF supports sparse (zeros) input but not missing values, so proteomics matrices require careful missing-value handling before OD modeling
- Feature selection operates on normalized data, so results depend on the normalization applied per dataset

## Evidence

- [readme] mosaicMPI integration scope across omics modalities: [evidence span withheld — anchor status: external_doc]
