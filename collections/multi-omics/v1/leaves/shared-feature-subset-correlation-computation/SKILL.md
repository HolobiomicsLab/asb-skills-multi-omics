---
name: shared-feature-subset-correlation-computation
description: Use when integrating factorized program matrices (e.g. cNMF outputs in
  .h5ad) across datasets with incomplete feature overlap, where program-pair correlations
  must be computed over shared features before anchoring.
when_to_use_negative:
- Input is a raw expression/count matrix rather than factorized program feature-score
  matrices — correlation should be run on programs, not individual features
- Datasets share all samples and features and are analyzed jointly, so no mosaic integration
  or cross-dataset anchoring is needed
- Anchor edges already exist (anchor_edge_list.tsv from a prior run) and only downstream
  community detection or metadata propagation is required
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: mosaicmpi
  role: Provides the anchoring workflow that computes program-pair correlations over
    shared features and applies dynamic thresholding to select anchors
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: NumPy
  role: Stores the Pearson correlation matrix of all programs within and between datasets
    as the NumPy-compatible pearson.df.npz file
- name: pandas
  role: Holds the program feature-score matrices (from pandas data matrices or AnnData
    files) during pairwise correlation
- name: AnnData
  role: Import format for the per-dataset program feature-score .h5ad objects used
    as correlation inputs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:2aa416cc78bff2136e9b1a5f68eced68d44aed9c3273a41b4b18f11237ab4a15
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
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Shared-feature-subset correlation computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes Pearson correlations between factorized program feature-score matrices within and across datasets, restricting each comparison to the shared features between the two datasets, as the basis for mosaicMPI's dynamic-threshold program anchoring.

## When to use

Use when integrating factorized program matrices (e.g. cNMF outputs in .h5ad) across datasets with incomplete feature overlap, where program-pair correlations must be computed over shared features before anchoring.

## When NOT to use

- Input is a raw expression/count matrix rather than factorized program feature-score matrices — correlation should be run on programs, not individual features
- Datasets share all samples and features and are analyzed jointly, so no mosaic integration or cross-dataset anchoring is needed
- Anchor edges already exist (anchor_edge_list.tsv from a prior run) and only downstream community detection or metadata propagation is required

## Inputs

- Factorized program feature-score matrices for each dataset (results/<dataset>/<dataset>.h5ad, AnnData format)
- pandas data matrices or AnnData objects of program feature scores from cNMF factorization

## Outputs

- pearson.df.npz: NumPy-compatible Pearson correlation matrix of all programs within and between datasets
- pairwise_corr_thresholds.txt: 95th-percentile dynamic thresholds per dataset and dataset pair
- anchor_edge_list.tsv: list of anchor program pairs exceeding their dataset-pair threshold

## How to apply

Load the factorized program feature-score matrices for each dataset (task_002 outputs, e.g. results/<dataset>/<dataset>.h5ad) using mosaicmpi with AnnData/pandas objects. For every within- and between-dataset program pair, restrict the comparison to the features shared by the two datasets and compute the Pearson correlation, writing the full matrix to pearson.df.npz in NumPy-compatible format. Because datasets differ systematically, build an empirical null from the mirrored negative part of each correlation distribution and take its 95th percentile as a per-dataset and per-dataset-pair threshold, recording values in pairwise_corr_thresholds.txt. Keep only program pairs whose correlation exceeds their dataset-pair threshold and write these anchor edges to anchor_edge_list.tsv; on the three CPTAC modalities this should retain ~50,424 anchor pairs (~6.1% of all pairwise correlations).

## Related tools

- **mosaicmpi** (Provides the anchoring workflow that computes program-pair correlations over shared features and applies dynamic thresholding to select anchors) — https://github.com/MorrissyLab/mosaicMPI
- **NumPy** (Stores the Pearson correlation matrix of all programs within and between datasets as the NumPy-compatible pearson.df.npz file)
- **pandas** (Holds the program feature-score matrices (from pandas data matrices or AnnData files) during pairwise correlation)
- **AnnData** (Import format for the per-dataset program feature-score .h5ad objects used as correlation inputs)

## Examples

```
import numpy as np; corr = np.load("pearson.df.npz")
```

## Evaluation signals

- pearson.df.npz loads with NumPy (np.load) and covers all programs within and between the datasets
- Each threshold in pairwise_corr_thresholds.txt equals the 95th percentile of the mirrored negative-correlation null for its dataset/dataset-pair
- Every edge in anchor_edge_list.tsv has a correlation exceeding its dataset-pair threshold; no below-threshold pairs remain
- Over the three CPTAC modalities the anchor edge list contains approximately 50,424 anchor pairs (~6.1% of all pairwise correlations)

## Limitations

- Mosaic integration may lose relevant information between modalities because not all features are shared between datasets
- Systematic between-dataset differences require per-dataset-pair threshold calibration; a single global correlation cutoff is inappropriate
- cNMF/mosaicMPI inputs support sparse (zeros) data but not missing values

## Evidence
