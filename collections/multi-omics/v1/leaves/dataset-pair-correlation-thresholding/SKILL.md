---
name: dataset-pair-correlation-thresholding
description: Use when integrating cNMF-derived molecular programs across cohorts or
  -omics modalities with incomplete feature overlap, and you need dataset-pair-specific
  correlation thresholds to select highly correlated program pairs as anchors.
when_to_use_negative:
- Datasets already share all features and samples, so no mosaic/pair-specific threshold
  calibration is needed
- The goal is within-dataset program discovery (cNMF factorization) rather than cross-dataset
  program anchoring
- Data contains missing values — cNMF/mosaicMPI factorization supports sparse (zeros)
  data but not missing values, a prerequisite before correlation
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: mosaicMPI
  role: Provides the anchoring module implementing the dynamic-threshold scheme and
    workflow subcommands for correlating programs across datasets
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: AnnData
  role: Input format for the factorized program feature-score matrices (.h5ad) loaded
    before correlation
- name: pandas
  role: Handles the program feature-score data matrices used for pairwise Pearson
    correlation
- name: NumPy
  role: Storage format for the pearson.df.npz correlation matrix output
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:99772965593c761f17ab5d0c3505e3f27354f37775b4d290abd3add78ace8150
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
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# dataset-pair-correlation-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A dynamic-thresholding scheme from mosaicMPI that converts Pearson correlations between program pairs (within and between datasets) into anchor edges by calibrating a per-dataset-pair threshold from an empirical null distribution built from the mirrored negative part of the correlation distribution.

## When to use

Use when integrating cNMF-derived molecular programs across cohorts or -omics modalities with incomplete feature overlap, and you need dataset-pair-specific correlation thresholds to select highly correlated program pairs as anchors.

## When NOT to use

- Datasets already share all features and samples, so no mosaic/pair-specific threshold calibration is needed
- The goal is within-dataset program discovery (cNMF factorization) rather than cross-dataset program anchoring
- Data contains missing values — cNMF/mosaicMPI factorization supports sparse (zeros) data but not missing values, a prerequisite before correlation

## Inputs

- Factorized program feature-score matrices per dataset (AnnData .h5ad files, e.g. results/<dataset>/<dataset>.h5ad)
- CPTAC-style multi-modality datasets (e.g. cptac_snrna.h5ad) with incomplete feature overlap

## Outputs

- pearson.df.npz — NumPy-compatible Pearson correlation matrix of all programs within and between datasets
- pairwise_corr_thresholds.txt — 95th-percentile dynamic thresholds per dataset and dataset pair
- anchor_edge_list.tsv — edge list of program pairs whose correlation exceeds their dataset-pair threshold

## How to apply

Load the factorized program feature-score matrices for each dataset (e.g. results/<dataset>/<dataset>.h5ad via mosaicmpi/AnnData and pandas). For every within- and between-dataset program pair, restrict the comparison to the shared features between the two datasets and compute Pearson correlations, storing the full matrix in NumPy-compatible format as pearson.df.npz. For each dataset and dataset pair, construct an empirical null distribution of correlations from the mirrored negative correlations and take its 95th percentile as the threshold, recording values in pairwise_corr_thresholds.txt; this per-pair calibration accounts for systematic between-dataset differences. Keep program pairs whose correlation exceeds their dataset-pair threshold as anchor edges and write the edge list to anchor_edge_list.tsv.

## Related tools

- **mosaicMPI** (Provides the anchoring module implementing the dynamic-threshold scheme and workflow subcommands for correlating programs across datasets) — https://github.com/MorrissyLab/mosaicMPI
- **AnnData** (Input format for the factorized program feature-score matrices (.h5ad) loaded before correlation)
- **pandas** (Handles the program feature-score data matrices used for pairwise Pearson correlation)
- **NumPy** (Storage format for the pearson.df.npz correlation matrix output)

## Examples

```
import pandas as pd, numpy as np; shared1, shared2 = p1.loc[shared], p2.loc[shared]; pearson = shared1.T.corr(shared2.T); np.savez('pearson.df.npz', pearson)
```

## Evaluation signals

- pairwise_corr_thresholds.txt contains a 95th-percentile value for every dataset and dataset pair, derived from the mirrored negative part of the correlation distribution
- Over the three CPTAC modalities, the anchor_edge_list.tsv contains approximately 50,424 anchor pairs (~6.1% of all pairwise correlations)
- pearson.df.npz is loadable via NumPy and covers all programs within and between datasets, restricted to shared features for between-dataset pairs
- Every retained anchor edge has a correlation strictly exceeding its dataset-pair threshold

## Limitations

- Mosaic integration may lose relevant information between modalities because not all features are shared
- Systematic between-dataset differences require calibrating a separate threshold per dataset and dataset pair rather than one global cutoff
- No software version information is stated in the article for the tools/methods

## Evidence

- [readme] cNMF program discovery prerequisite: [evidence span withheld — anchor status: external_doc]
