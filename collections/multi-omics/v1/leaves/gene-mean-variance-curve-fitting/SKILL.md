---
name: gene-mean-variance-curve-fitting
description: Use when identifying overdispersed/highly-variable features from normalized
  count or intensity matrices across modalities (bulk RNA-Seq, proteomics, snRNA-Seq)
  where a modality-specific Poisson model does not apply.
when_to_use_negative:
- When a curated feature list already exists — use mosaicmpi select-hvf custom instead
  of refitting the mean-variance curve
- When data contain missing values or negative values that cannot be removed — mosaicMPI
  input rules reject these before overdispersion modeling
- When a single scRNA-Seq dataset is being analyzed under cNMF's own calibrated model
  — use the cnmf select-hvf method rather than the general curve fit
edam_operation: http://edamontology.org/operation_3226
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: mosaicMPI
  role: Provides select-hvf (stdeconvolve method) to fit the mean-variance dependence
    of normalized data and compute OD scores; check-h5ad validates inputs first
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Used to load the per-dataset count matrices (bulk RNA-Seq, proteomics, snRNA-Seq)
    as data frames for AnnData construction
- name: AnnData
  role: Input file/object format for the h5ad matrices passed to mosaicmpi select-hvf
- name: NumPy
  role: Numeric backend for variance modeling on the normalized matrices
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:b87a7a2f9349a1707d6cbc47b58d257845634ad7a1c3e7bb47bf27035a215c31
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
  edam_operation: http://edamontology.org/operation_3226
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# gene-mean-variance-curve-fitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generalizable overdispersed feature selection via mean-variance curve fitting: a smooth curve is fit to the mean-variance dependence of normalized data and residual variance (OD score) is used to select overdispersed features, replacing cNMF's scRNA-Seq-calibrated Poisson model for modalities like mass spectrometry proteomics.

## When to use

Use when identifying overdispersed/highly-variable features from normalized count or intensity matrices across modalities (bulk RNA-Seq, proteomics, snRNA-Seq) where a modality-specific Poisson model does not apply.

## When NOT to use

- When a curated feature list already exists — use mosaicmpi select-hvf custom instead of refitting the mean-variance curve
- When data contain missing values or negative values that cannot be removed — mosaicMPI input rules reject these before overdispersion modeling
- When a single scRNA-Seq dataset is being analyzed under cNMF's own calibrated model — use the cnmf select-hvf method rather than the general curve fit

## Inputs

- Normalized count/intensity matrix per dataset (e.g. h5ad/AnnData or pandas data matrix) from bulk RNA-Seq STAR counts, PDC000204 proteomics, or snRNA-Seq feature counts
- Optional whitespace-delimited custom feature list for select-hvf custom
- Optional metadata table for stratification (e.g. patient)

## Outputs

- overdispersed_genes.csv with per-dataset selected gene lists
- feature_mean_var.pdf (mean-variance relationship with fitted model and selected features in red)
- feature_odscore.pdf (overdispersion score histogram)

## How to apply

Load each dataset (e.g. CPTAC-3 GBM bulk RNA-Seq STAR counts, PDC000204 proteomics, snRNA-Seq feature counts) via pandas/AnnData and remove invalid features (missing values, negative values, zero variance) as checked by mosaicmpi check-h5ad, since cNMF supports sparse zeros but not missing values. Fit a smooth curve to the mean-variance dependence of the normalized data using the STdeconvolve method (Miller, et al. Nat. Comm. 2022) as reimplemented in mosaicMPI's select-hvf. Compute the overdispersion (OD) score as the residual variance after correcting for the per-dataset mean-variance relationship, then select features with OD score > 1 (optionally stratifying by metadata such as patient and taking the union via select-hvf --stratify_by). Generate diagnostic plots feature_mean_var.pdf (model fit and selected features in red) and feature_odscore.pdf (OD score histogram), and write the selected features to overdispersed_genes.csv. Rationale: mosaicMPI needs program discovery on all available features per dataset, so overdispersion must be defined per-dataset rather than with a single modality-calibrated model.

## Related tools

- **mosaicMPI** (Provides select-hvf (stdeconvolve method) to fit the mean-variance dependence of normalized data and compute OD scores; check-h5ad validates inputs first) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Used to load the per-dataset count matrices (bulk RNA-Seq, proteomics, snRNA-Seq) as data frames for AnnData construction)
- **AnnData** (Input file/object format for the h5ad matrices passed to mosaicmpi select-hvf)
- **NumPy** (Numeric backend for variance modeling on the normalized matrices)

## Examples

```
mosaicmpi select-hvf stdeconvolve -n cptac_snrna.h5ad -o select_hvf_results
```

## Evaluation signals

- overdispersed_genes.csv exists, is parseable, and per-dataset gene counts approximate the reported 8454 (bulk RNA), 4541 (protein), and 7072 (snRNA) genes
- feature_mean_var.pdf shows the fitted mean-variance model with selected features highlighted in red
- feature_odscore.pdf shows an OD score histogram and all retained features have OD score > 1
- No invalid features (missing values, negative values, zero variance) remain in the input per mosaicmpi check-h5ad
- Selected features are obtained per-dataset on normalized data rather than with cNMF's Poisson model

## Limitations

- OD score > 1 is a default relaxed threshold and may require tuning per dataset
- The cnmf select-hvf method is calibrated for scRNA-Seq-style sparse count data, not for proteomics or other normalized modalities
- No software version information is stated in the article; check mosaicmpi --help for the installed version

## Evidence

- [other] mosaicMPI selects overdispersed genes using an overdispersion score (OD score), defined as the residual variance after correcting for the per-dataset mean-variance relationship: [evidence span withheld — anchor status: not_found]
- [other] Fit a smooth curve to the mean-variance dependence of the normalized data for each dataset, following the STdeconvolve method: [evidence span withheld — anchor status: not_found]
