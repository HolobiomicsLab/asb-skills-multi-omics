---
name: stdeconvolve-overdispersion-modeling
description: Use when the input is a normalized count-like matrix (e.g. CPTAC-3 GBM
  bulk RNA-Seq STAR counts, PDC000204 proteomics, or snRNA-Seq feature counts loaded
  as .h5ad/AnnData) and overdispersed features must be selected before factorization,
  without relying on cNMF's Poisson assumption.
when_to_use_negative:
- The data are raw, unnormalized counts where a mean-variance dependence has not yet
  been modeled — normalization must precede select-hvf.
- A known custom feature set already exists — use `mosaicmpi select-hvf custom` with
  a whitespace-delimited feature file instead of curve fitting.
- The dataset has missing values if it will feed cNMF downstream — cNMF supports sparse
  (zero-containing) data but not missing values, so missing values must be resolved
  first.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3361
tools:
- name: mosaicMPI
  role: Provides the select-hvf subcommand implementing the stdeconvolve mean-variance
    curve-fitting method and OD-score-based feature selection
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Loads count matrices from which AnnData objects are made for variance modeling
- name: AnnData
  role: File/object format (.h5ad) for holding normalized matrices and per-dataset
    selected feature annotations
- name: NumPy
  role: Numerical backend supporting correlation/program matrices within the mosaicMPI
    workflow
- name: cNMF
  role: Downstream factorization that uses the over-dispersed feature subset and NNLS
    refits programs across all features; its Poisson model is the baseline this skill
    generalizes beyond
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:643c679ffd52367bd36262fe6e7a8d8b5ee6b261bb61bbe678cf874294f4c885
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# stdeconvolve-overdispersion-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies overdispersed (highly variable) features by fitting a smooth curve to the mean-variance dependence of normalized data using the STdeconvolve method (Miller et al., Nat. Comm. 2022) as implemented in mosaicMPI's select-hvf, generalizing feature selection beyond cNMF's scRNA-Seq-calibrated Poisson model to modalities such as bulk RNA-Seq and mass spectrometry proteomics.

## When to use

Use when the input is a normalized count-like matrix (e.g. CPTAC-3 GBM bulk RNA-Seq STAR counts, PDC000204 proteomics, or snRNA-Seq feature counts loaded as .h5ad/AnnData) and overdispersed features must be selected before factorization, without relying on cNMF's Poisson assumption.

## When NOT to use

- The data are raw, unnormalized counts where a mean-variance dependence has not yet been modeled — normalization must precede select-hvf.
- A known custom feature set already exists — use `mosaicmpi select-hvf custom` with a whitespace-delimited feature file instead of curve fitting.
- The dataset has missing values if it will feed cNMF downstream — cNMF supports sparse (zero-containing) data but not missing values, so missing values must be resolved first.

## Inputs

- Normalized count matrix per dataset (bulk RNA-Seq STAR counts, PDC000204 proteomics, snRNA-Seq feature counts) as pandas DataFrame or AnnData .h5ad
- Optional metadata table for stratified HVF selection (e.g. patient annotations)
- Optional custom whitespace-delimited feature list for select-hvf custom

## Outputs

- overdispersed_genes.csv — per-dataset selected gene/feature lists
- feature_mean_var.pdf — mean-variance curve with model and selected features highlighted in red
- feature_odscore.pdf — overdispersion score histogram
- Dataset .h5ad annotated with selected over-dispersed (highly variable) features for downstream factorization

## How to apply

Load each dataset with pandas/AnnData and remove invalid features (missing values, negative values, zero variance) — cNMF-based inputs may be sparse with zeros but not missing values. Fit a smooth curve to the mean-variance dependence of the normalized data per dataset with the `stdeconvolve` method in `mosaicmpi select-hvf`, then compute an overdispersion (OD) score per feature from the model residuals. Select features with OD score > 1 (default relaxed threshold), optionally stratifying selection by metadata such as patient via `select-hvf --stratify_by` and taking the union. Diagnostic outputs feature_mean_var.pdf (selected features highlighted in red) and feature_odscore.pdf (OD score histogram) plus overdispersed_genes.csv support QC before program factorization, where the over-dispersed feature subset is used for cNMF factorization and NNLS re-fits programs across all features.

## Related tools

- **mosaicMPI** (Provides the select-hvf subcommand implementing the stdeconvolve mean-variance curve-fitting method and OD-score-based feature selection) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Loads count matrices from which AnnData objects are made for variance modeling)
- **AnnData** (File/object format (.h5ad) for holding normalized matrices and per-dataset selected feature annotations)
- **NumPy** (Numerical backend supporting correlation/program matrices within the mosaicMPI workflow)
- **cNMF** (Downstream factorization that uses the over-dispersed feature subset and NNLS refits programs across all features; its Poisson model is the baseline this skill generalizes beyond)

## Examples

```
mosaicmpi select-hvf stdeconvolve -n cptac_snrna.h5ad
```

## Evaluation signals

- overdispersed_genes.csv exists, parses cleanly, and per-dataset selected feature counts approximate the reported 8454 (bulk RNA), 4541 (protein), and 7072 (snRNA) genes
- feature_mean_var.pdf shows a fitted smooth mean-variance curve with selected features highlighted in red
- feature_odscore.pdf shows an OD score histogram consistent with a threshold of OD score > 1
- No invalid features remain: all retained features have non-missing, non-negative values and non-zero variance (mosaicmpi check-h5ad passes)
- The stdeconvolve method was applied to normalized (not raw) data, matching the select-hvf default behavior

## Limitations

- Mosaic integration can lose relevant information between modalities because not all features are shared across datasets.
- Combining distinct data types in the same statistical framework is not generalizable, so each modality requires bespoke pre-processing (e.g. normalization and variance modeling) before integration.
- cNMF-based handling only supports sparse data (with zeros), not missing values; no software version information is stated in the article methods.

## Evidence

- [readme] mosaicMPI is available at https://github.com/MorrissyLab/mosaicMPI.: [evidence span withheld — anchor status: external_doc]
