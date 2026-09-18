---
name: program-anchor-edge-list-construction
description: Use when factorized program matrices (e.g. .h5ad from task_002 cNMF runs
  over CPTAC datasets) must be linked into anchor edges for mosaic integration, with
  per-dataset-pair correlation thresholds.
when_to_use_negative:
- Datasets share all samples and an identical, complete feature set, so a simpler
  joint-factorization or intersection-based integration is possible
- Programs have not yet been factorized (no cNMF feature-score matrices available)
  — run the factorization step first
- The goal is feature-level integration rather than program-level anchoring; mosaicMPI
  explicitly avoids subsetting genes to an intersection or highly-variable subset
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mosaicMPI
  role: Provides the anchoring framework and workflow subcommands for computing program
    correlations and building the anchor edge list
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: NumPy
  role: Storage format target for the Pearson correlation matrix (pearson.df.npz)
- name: pandas
  role: Handling of program feature-score data matrices loaded into AnnData objects
- name: AnnData
  role: Input file/object format for the factorized program matrices
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:8ba3e0a7c0d0bf13d77fbd8070fd10a2e8fcaa435007f629c73bcd60f6b36320
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# program-anchor-edge-list-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs the anchoring program-correlation network for mosaicMPI by computing Pearson correlations between cNMF-derived program feature-score matrices within and across datasets, then retaining program pairs whose correlation exceeds a dynamically calibrated 95th-percentile threshold as anchor edges.

## When to use

Use when factorized program matrices (e.g. .h5ad from task_002 cNMF runs over CPTAC datasets) must be linked into anchor edges for mosaic integration, with per-dataset-pair correlation thresholds.

## When NOT to use

- Datasets share all samples and an identical, complete feature set, so a simpler joint-factorization or intersection-based integration is possible
- Programs have not yet been factorized (no cNMF feature-score matrices available) — run the factorization step first
- The goal is feature-level integration rather than program-level anchoring; mosaicMPI explicitly avoids subsetting genes to an intersection or highly-variable subset

## Inputs

- Factorized program feature-score matrices for each dataset as AnnData .h5ad files (results/<dataset>/<dataset>.h5ad)
- Program-to-dataset assignment (which programs belong to which of the three CPTAC datasets)

## Outputs

- pearson.df.npz — Pearson correlation matrix of all programs within and between datasets, NumPy-compatible
- pairwise_corr_thresholds.txt — 95th-percentile null-derived correlation thresholds per dataset and dataset pair
- anchor_edge_list.tsv — list of anchor program pairs whose correlations exceed their dataset-pair threshold

## How to apply

Load the factorized program feature-score matrices (results/<dataset>/<dataset>.h5ad) with mosaicmpi/AnnData and pandas. For each within- and between-dataset program pair, restrict the comparison to the shared features between the two datasets (mosaic integration tolerates incomplete feature overlap) and compute Pearson correlations, storing the matrix as pearson.df.npz in NumPy-compatible format. Build an empirical null distribution of correlations from the mirrored negative part of the correlation distribution and take its 95th percentile as the threshold for each dataset and dataset pair, calibrating for systematic between-dataset differences; record thresholds in pairwise_corr_thresholds.txt. Keep program pairs whose correlation exceeds their dataset-pair threshold as anchor edges and write the edge list to anchor_edge_list.tsv.

## Related tools

- **mosaicMPI** (Provides the anchoring framework and workflow subcommands for computing program correlations and building the anchor edge list) — https://github.com/MorrissyLab/mosaicMPI
- **NumPy** (Storage format target for the Pearson correlation matrix (pearson.df.npz))
- **pandas** (Handling of program feature-score data matrices loaded into AnnData objects)
- **AnnData** (Input file/object format for the factorized program matrices)

## Evaluation signals

- The anchor edge list over the three CPTAC modalities contains approximately 50,424 anchor pairs (~6.1% of all pairwise correlations)
- pearson.df.npz loads as a NumPy-compatible correlation matrix covering all programs within and between datasets
- pairwise_corr_thresholds.txt contains a 95th-percentile threshold for each dataset and each dataset pair, derived from the mirrored-negative null distribution
- Every retained edge in anchor_edge_list.tsv has a correlation exceeding its corresponding dataset-pair threshold

## Limitations

- A potential concern with mosaic integration is loss of relevant information between modalities given that not all features are shared
- The mirrored-negative null assumes the negative side of the correlation distribution approximates the null, which may not hold for skewed correlation distributions
- Thresholds are calibrated per dataset pair to handle systematic between-dataset differences; poorly matched modalities may yield overly strict or lenient thresholds

## Evidence

- [readme] README: mismatched features supported: [evidence span withheld — anchor status: external_doc]
