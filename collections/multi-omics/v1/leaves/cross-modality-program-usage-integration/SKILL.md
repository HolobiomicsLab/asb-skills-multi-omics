---
name: cross-modality-program-usage-integration
description: Use when per-patient program/community usage values exist in ≥2 -omics
  modalities (e.g., RNA and protein in CPTAC GBM) and the goal is to test whether
  combined cross-modality usage reveals clinical (survival) associations.
when_to_use_negative:
- Only one modality is available — the reference finding shows RNA-only or protein-only
  stratifications of CLA/MES usage fail to reveal the survival difference
- Datasets lack program-level deconvolution (cNMF) and mosaicMPI integration; raw
  feature-level data cannot be directly stratified by community usage
- No matched clinical/overall-survival annotations are available to propagate onto
  the integrated program network
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: mosaicMPI
  role: Provides the integrated program network and cross-modality community usage
    values used to define the co-usage vs classical-only stratification
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Merging stratification labels with CPTAC clinical survival metadata and handling
    AnnData-backed data matrices
- name: NumPy
  role: Numerical handling of program usage matrices and correlation structures (e.g.,
    pearson.df.npz outputs)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:47fe609a8591cbfd966cb3e0c13fae2b0b330943e04e1f23b5511ad002b5c4a9
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
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cross-modality-program-usage-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratifies patients by joint usage of program communities (e.g., classical CLA and mesenchymal MES) measured across both RNA and protein modalities, then tests survival differences by Kaplan-Meier and log-rank. Single-modality stratifications alone can miss survival signals that only emerge when RNA and protein program usage are combined.

## When to use

Use when per-patient program/community usage values exist in ≥2 -omics modalities (e.g., RNA and protein in CPTAC GBM) and the goal is to test whether combined cross-modality usage reveals clinical (survival) associations.

## When NOT to use

- Only one modality is available — the reference finding shows RNA-only or protein-only stratifications of CLA/MES usage fail to reveal the survival difference
- Datasets lack program-level deconvolution (cNMF) and mosaicMPI integration; raw feature-level data cannot be directly stratified by community usage
- No matched clinical/overall-survival annotations are available to propagate onto the integrated program network

## Inputs

- mosaicMPI network integration pickle (network_integration.pkl.gz) containing datasets, program correlations, metadata, and program usage
- Per-patient community usage values for CLA (C1) and MES communities in RNA and protein datasets
- CPTAC clinical survival metadata (overall survival times and censoring indicators)

## Outputs

- Kaplan-Meier survival plot (kaplan_meier_figure.png) comparing co-usage vs classical-only groups
- Log-rank test statistic and p-value (logrank_test), expected ≈ 0.0123
- Patient stratification labels (co-usage vs classical-only)

## How to apply

Load the mosaicMPI network integration (e.g., network_integration.pkl.gz from task_004), which contains datasets, correlations, metadata, and program usage. Extract per-patient community usage for the classical (C1/CLA) and mesenchymal (MES) program communities separately in the RNA and protein datasets. Define a co-usage group (high usage of both classical and mesenchymal programs in both modalities) versus a classical-only group, then merge these labels with CPTAC overall survival times and censoring indicators. Fit Kaplan-Meier curves per group and compute the log-rank test on hazards; in the reference study, the co-usage versus classical-only comparison yielded p = 0.0123 (Figure 4E). The rationale is that mosaicMPI integrates programs across modalities without subsetting to shared features or shared sample IDs, so community usage is comparable across RNA and protein even for mismatched feature spaces.

## Related tools

- **mosaicMPI** (Provides the integrated program network and cross-modality community usage values used to define the co-usage vs classical-only stratification) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Merging stratification labels with CPTAC clinical survival metadata and handling AnnData-backed data matrices)
- **NumPy** (Numerical handling of program usage matrices and correlation structures (e.g., pearson.df.npz outputs))

## Examples

```
import pickle; net = pickle.load(open('network_integration.pkl.gz','rb')); usage = net.get_usage(); labels = (usage['rna']['CLA'] > t) & (usage['rna']['MES'] > t) & (usage['protein']['CLA'] > t) & (usage['protein']['MES'] > t)
```

## Evaluation signals

- Log-rank p-value for co-usage vs classical-only hazards should approximate the reported 0.0123 (Figure 4E)
- Both output files (kaplan_meier_figure.png and logrank_test) must exist with correct formats
- Stratification must use usage from BOTH RNA and protein modalities; verify single-modality-only groupings do not reproduce the significant result
- Kaplan-Meier curves should show worse survival for the CLA+MES co-usage group relative to the CLA-only group
- Clinical metadata merge should preserve all patients with available overall survival times and censoring indicators

## Limitations

- Mosaic integration can lose relevant information between modalities because not all features are shared across RNA and protein datasets
- cNMF supports sparse (zero-containing) input data but not missing values
- Clinical annotation propagation is statistical, so per-patient co-usage assignments depend on the integrated network resolution and community definitions

## Evidence

- [readme] README: multi-omics integration without shared sample IDs: [evidence span withheld — anchor status: external_doc]
