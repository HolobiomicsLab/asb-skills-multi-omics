---
name: mirrored-negative-correlation-null-estimation
description: Use when constructing an anchor edge list from pairwise program correlation
  matrices across CPTAC-style datasets where feature overlap is incomplete and a dataset-pair-specific
  correlation cutoff is needed.
when_to_use_negative:
- Datasets share all samples and features, so direct sample-level integration (e.g.
  correlation on shared samples) is possible instead of program-level anchoring
- A prior threshold or validated cutoff for program-pair correlations already exists
  for the dataset pair being integrated
- Programs have not yet been factorized — run cNMF program discovery first; there
  are no program feature scores to correlate
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: mosaicmpi
  role: Provides the anchoring workflow and AnnData loading used to compute program
    correlations and apply the dynamic threshold
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: NumPy
  role: Storage format target for the pearson.df.npz correlation matrix consumed by
    the null/threshold estimation
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Handles the program feature-score data frames used to compute shared-feature
    Pearson correlations
- name: AnnData
  role: File/object format from which the factorized program feature-score matrices
    are imported
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:70a3915782cac5e69e16cc36a7964b8823e57d0a1d5d186210b1861996af1e8b
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
  - mosaicmpi
  - pandas
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Mirrored-negative correlation null estimation for program anchoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

mosaicMPI's dynamic thresholding scheme that builds an empirical null distribution of program-program Pearson correlations from the mirrored negative part of the correlation distribution, taking its 95th percentile as the threshold for calling anchoring program pairs within and between datasets.

## When to use

Use when constructing an anchor edge list from pairwise program correlation matrices across CPTAC-style datasets where feature overlap is incomplete and a dataset-pair-specific correlation cutoff is needed.

## When NOT to use

- Datasets share all samples and features, so direct sample-level integration (e.g. correlation on shared samples) is possible instead of program-level anchoring
- A prior threshold or validated cutoff for program-pair correlations already exists for the dataset pair being integrated
- Programs have not yet been factorized — run cNMF program discovery first; there are no program feature scores to correlate

## Inputs

- Factorized program feature-score matrices for each dataset as AnnData .h5ad files (e.g. results/<dataset>/<dataset>.h5ad from cNMF factorization)
- Program feature score tables readable from pandas data matrices or AnnData objects

## Outputs

- pearson.df.npz — pearson correlation matrix of all programs within and between datasets, compatible with NumPy
- pairwise_corr_thresholds.txt — 95th-percentile thresholds derived from the mirrored-negative null, per dataset and dataset pair
- anchor_edge_list.tsv — list of program pairs whose correlation exceeds their dataset-pair threshold

## How to apply

Load the factorized program feature-score matrices (results/<dataset>/<dataset>.h5ad) via mosaicmpi/AnnData and pandas. For each within- and between-dataset program pair, restrict the comparison to shared features between the two datasets, compute Pearson correlations, and store the full matrix as pearson.df.npz in NumPy-compatible format. Build the empirical null from the mirrored negative correlations and take its 95th percentile as the threshold, calibrated separately for each dataset and dataset pair to absorb systematic between-dataset differences; record thresholds in pairwise_corr_thresholds.txt. Keep program pairs whose correlation exceeds their dataset-pair threshold as anchor edges and write them to anchor_edge_list.tsv. Rationale: mosaic integration involves datasets with incomplete feature overlap and heterogeneous statistical properties, so a fixed global cutoff is not generalizable — the mirrored-negative null adapts per dataset pair.

## Related tools

- **mosaicmpi** (Provides the anchoring workflow and AnnData loading used to compute program correlations and apply the dynamic threshold) — https://github.com/MorrissyLab/mosaicMPI
- **NumPy** (Storage format target for the pearson.df.npz correlation matrix consumed by the null/threshold estimation) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Handles the program feature-score data frames used to compute shared-feature Pearson correlations)
- **AnnData** (File/object format from which the factorized program feature-score matrices are imported)

## Examples

```
mosaicmpi --help  # inspect subcommands; run the anchoring workflow after loading results/<dataset>/<dataset>.h5ad program matrices
```

## Evaluation signals

- anchor_edge_list.tsv over the three CPTAC modalities contains approximately 50,424 anchor pairs (~6.1% of all pairwise correlations)
- pairwise_corr_thresholds.txt contains a 95th-percentile threshold for every dataset and dataset pair, with between-dataset pairs calibrated separately
- pearson.df.npz loads in NumPy and covers all programs within and between datasets
- Every retained anchor edge's correlation exceeds its corresponding dataset-pair threshold
- Within-dataset correlations were computed on all shared features; between-dataset correlations restricted to the shared feature subset

## Limitations

- Mosaic integration may lose relevant information between modalities because not all features are shared, so between-dataset correlations rest on restricted feature subsets
- Calibration for systematic between-dataset differences is empirical; datasets with skewed correlation distributions may yield unstable 95th-percentile thresholds

## Evidence

- [other] Mirrored-negative null and 95th-percentile threshold: [evidence span withheld — anchor status: not_found]
- [readme] Mismatched-feature integration motivation: [evidence span withheld — anchor status: external_doc]
