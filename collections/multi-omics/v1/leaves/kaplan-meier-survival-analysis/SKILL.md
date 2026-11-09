---
name: kaplan-meier-survival-analysis
description: Use when single-modality (RNA-only or protein-only) program-usage stratification
  of CPTAC GBM patients shows no survival differences and a combined-modality classical+mesenchymal
  co-usage stratification is needed.
when_to_use_negative:
- When the analysis goal is cross-cohort or cross-modality program integration itself
  (deconvolution, network integration, or metadata propagation) rather than downstream
  survival testing on already-propagated usage values.
- When only one modality's program usage is available and no combined RNA+protein
  stratification is intended, since the reported finding depends on cross-modality
  co-usage.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_0622
tools:
- name: mosaicMPI
  role: Provides the integrated network landscape and propagates clinical/molecular
    annotations onto program communities, supplying per-patient usage values used
    for survival stratification.
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Handles per-patient usage tables and merging of stratification labels with
    CPTAC survival metadata (pandas data matrices can be imported into AnnData objects).
- name: NumPy
  role: Numeric handling of usage values; mosaicMPI exports pearson correlation matrices
    in a NumPy-compatible .npz format.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:bffdc933a703a86f2adaddfc98698fa578d8b432d3caa06bb01fcdef64e30eb3
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
  - NumPy
  - mosaicMPI
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# kaplan-meier-survival-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratifies patients into survival groups based on cross-modality (RNA + protein) usage of classical (C1/CLA) and mesenchymal (MES) program communities from the mosaicMPI network integration, then compares Kaplan-Meier survival curves with a log-rank test to reveal co-usage-associated survival differences invisible in single-modality stratification.

## When to use

Use when single-modality (RNA-only or protein-only) program-usage stratification of CPTAC GBM patients shows no survival differences and a combined-modality classical+mesenchymal co-usage stratification is needed.

## When NOT to use

- When the analysis goal is cross-cohort or cross-modality program integration itself (deconvolution, network integration, or metadata propagation) rather than downstream survival testing on already-propagated usage values.
- When only one modality's program usage is available and no combined RNA+protein stratification is intended, since the reported finding depends on cross-modality co-usage.

## Inputs

- network_integration.pkl.gz — pickled mosaicMPI network integration (datasets, correlations, metadata, program usage) from task_004
- Per-patient community usage values for classical (C1) and mesenchymal program communities in the RNA dataset
- Per-patient community usage values for the same communities in the protein (proteomics) dataset
- CPTAC clinical survival metadata (overall survival times and censoring indicators)

## Outputs

- kaplan_meier_figure.png — Kaplan-Meier survival curves for the co-usage versus classical-only groups
- logrank_test — log-rank statistic and p-value comparing hazards between groups

## How to apply

Load the previously generated mosaicMPI network integration (network_integration.pkl.gz from task_004), which contains datasets, correlations, metadata, and program usage. Extract per-patient usage values for the classical (C1) and mesenchymal program communities in both the RNA and protein datasets, then define a co-usage group (high usage of both programs in both modalities) versus a classical-only group. Merge these labels with CPTAC clinical overall-survival times and censoring indicators, fit Kaplan-Meier curves per group, and compute the log-rank hazard comparison. The rationale is that mosaicMPI's statistical propagation of clinical annotations onto the integrated program network enables sample-level metadata transfer across modalities, so survival grouping can leverage community usage from RNA and proteomics simultaneously rather than each modality separately.

## Related tools

- **mosaicMPI** (Provides the integrated network landscape and propagates clinical/molecular annotations onto program communities, supplying per-patient usage values used for survival stratification.) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Handles per-patient usage tables and merging of stratification labels with CPTAC survival metadata (pandas data matrices can be imported into AnnData objects).)
- **NumPy** (Numeric handling of usage values; mosaicMPI exports pearson correlation matrices in a NumPy-compatible .npz format.)

## Examples

```
import pickle; integ = pickle.load(open('network_integration.pkl.gz','rb'))  # extract C1/MES usage in RNA+protein, merge with CPTAC OS metadata, then log-rank test
```

## Evaluation signals

- The log-rank p-value comparing co-usage versus classical-only hazards is approximately 0.0123, matching the reported Figure 4E result.
- Both output files exist: kaplan_meier_figure.png (a rendered Kaplan-Meier plot) and logrank_test (containing the log-rank statistic and p-value).
- The stratification is based on high usage of BOTH classical and mesenchymal programs in BOTH RNA and protein modalities, not single-modality thresholds.
- Survival metadata merge preserves overall survival times and censoring indicators keyed to the correct CPTAC patient identifiers.
- Kaplan-Meier curves separate visually, with the CLA/MES co-usage group showing worse overall survival than the CLA-only group.

## Limitations

- A potential concern with mosaic integration is loss of relevant information between modalities given that not all features are shared, which can affect propagated usage values.
- The reported result is specific to the CPTAC GBM cohort and its clinical annotations; other cohorts may not replicate the p ≈ 0.0123 log-rank result.
- Single-modality stratifications (RNA-only or protein-only) of classical and mesenchymal usage failed to reveal survival differences, so this skill should not be expected to produce signal in single-modality settings.

## Evidence
