---
name: overdispersion-score-calculation
description: Use when feature selection for factorization is needed on normalized
  count or intensity matrices (bulk RNA-Seq, proteomics, snRNA-Seq) whose statistical
  properties do not fit cNMF's Poisson scRNA-Seq model, so overdispersed features
  must be found via mean-variance curve fitting per modality.
when_to_use_negative:
- The dataset's overdispersion model is already appropriate, i.e. data are sparse
  scRNA-Seq counts compatible with cNMF's Poisson model ('mosaicmpi select-hvf cnmf'
  may be preferred).
- A curated feature list already exists — use 'mosaicmpi select-hvf custom' with a
  whitespace-delimited features file instead of fitting the mean-variance curve.
- Input contains features with missing values, negative values, or zero variance that
  have not been filtered — these must be removed before overdispersion modeling.
edam_operation: http://edamontology.org/operation_3226
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
tools:
- name: mosaicMPI
  role: Provides the select-hvf subcommand that fits the mean-variance curve (stdeconvolve
    method) and computes OD scores with the OD > 1 threshold
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Used to load count matrices that can be converted to AnnData objects for input
- name: AnnData
  role: Container format for input matrices, importable by mosaicMPI (check-h5ad validates
    them)
- name: NumPy
  role: Array backend for variance modeling and residual computation on normalized
    matrices
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:f6e228ef376cd0ed0a9f3c192e53eec865d08c20ee8737fcc517b9b5a2a3e28b
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# overdispersion-score-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A modality-agnostic feature selection method from mosaicMPI that identifies overdispersed features (e.g. genes or proteins) by fitting a smooth curve to each dataset's mean-variance relationship and computing an overdispersion (OD) score from the residuals, selecting features with OD score > 1.

## When to use

Use when feature selection for factorization is needed on normalized count or intensity matrices (bulk RNA-Seq, proteomics, snRNA-Seq) whose statistical properties do not fit cNMF's Poisson scRNA-Seq model, so overdispersed features must be found via mean-variance curve fitting per modality.

## When NOT to use

- The dataset's overdispersion model is already appropriate, i.e. data are sparse scRNA-Seq counts compatible with cNMF's Poisson model ('mosaicmpi select-hvf cnmf' may be preferred).
- A curated feature list already exists — use 'mosaicmpi select-hvf custom' with a whitespace-delimited features file instead of fitting the mean-variance curve.
- Input contains features with missing values, negative values, or zero variance that have not been filtered — these must be removed before overdispersion modeling.

## Inputs

- Count or intensity matrices (e.g. bulk RNA-Seq STAR counts, mass spectrometry proteomics intensities, snRNA-Seq feature counts) as pandas DataFrames or AnnData .h5ad objects
- Optional whitespace-delimited custom feature list file (for select-hvf custom)
- Metadata (e.g. patient labels) for stratified selection

## Outputs

- overdispersed_genes.csv with per-dataset selected feature lists
- feature_mean_var.pdf diagnostic plot (mean-variance relationship with fitted model and selected features in red)
- feature_odscore.pdf diagnostic plot (overdispersion score histogram)

## How to apply

Load each count matrix (e.g. CPTAC-3 GBM bulk RNA-Seq STAR counts, PDC000204 proteomics, snRNA-Seq feature counts) with pandas/AnnData and produce a normalized matrix for variance modeling, handling missing values since cNMF-style sparse inputs do not tolerate them. Remove invalid features (missing values, negative values, zero variance) as checked by 'mosaicmpi check-h5ad' before modeling. Fit a smooth curve to the mean-variance dependence of the normalized data for each dataset using the STdeconvolve method (Miller, et al. Nat. Comm. 2022) reimplemented in 'mosaicmpi select-hvf' (rather than the 'cnmf' method), then compute each feature's OD score from the model residuals. Select features with OD score > 1 (the default relaxed threshold), optionally stratifying by metadata such as patient and taking the union via 'select-hvf --stratify_by'. Rationale: unlike cNMF's scRNA-Seq-calibrated Poisson model, residual variance after mean-variance correction generalizes overdispersion detection to proteomics and other normalized data types.

## Related tools

- **mosaicMPI** (Provides the select-hvf subcommand that fits the mean-variance curve (stdeconvolve method) and computes OD scores with the OD > 1 threshold) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Used to load count matrices that can be converted to AnnData objects for input)
- **AnnData** (Container format for input matrices, importable by mosaicMPI (check-h5ad validates them))
- **NumPy** (Array backend for variance modeling and residual computation on normalized matrices)

## Examples

```
mosaicmpi select-hvf -o cptac_snrna.h5ad --method stdeconvolve --stratify_by patient
```

## Evaluation signals

- overdispersed_genes.csv exists and is parseable, with per-dataset gene counts approximating the reported 8454 (bulk RNA), 4541 (protein), and 7072 (snRNA) genes for CPTAC-3 GBM.
- feature_mean_var.pdf shows the fitted mean-variance curve with selected overdispersed features highlighted in red.
- feature_odscore.pdf histogram shows the OD score distribution consistent with the OD > 1 cutoff.
- 'mosaicmpi check-h5ad' passes: no features with missing values, negative values, or zero variance remain before modeling.
- Selected feature sets are consistent across modalities despite heterogeneous statistical properties (bulk RNA-Seq, proteomics, snRNA-Seq).

## Limitations

- The OD > 1 threshold is a default relaxed cutoff; users may need to adjust it per modality.
- Potential loss of relevant information between modalities in mosaic integration because not all features are shared.
- cNMF-style sparse input handling does not tolerate missing values, so proteomics matrices with missing intensities must be treated per mosaicMPI's sparse-compatible input rules.
- No software version information is stated for the tools/methods discussed in the article.

## Evidence

- [full_text] mosaicMPI selects overdispersed genes using an overdispersion score (OD score), defined as the residual variance after correcting for the per-dataset mean-variance relationship, with a default relaxed threshold of OD score > 1.: [evidence span withheld — anchor status: not_found]
