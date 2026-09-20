---
name: community-usage-patient-stratification
description: Use when per-patient program-community usage (e.g., classical C1/CLA
  and mesenchymal MES) from an integrated mosaicMPI RNA+protein landscape must define
  survival groups tested by Kaplan-Meier/log-rank against CPTAC clinical metadata.
when_to_use_negative:
- 'Single-modality analysis is intended: RNA-only or protein-only stratifications
  of CPTAC GBM patients by classical and mesenchymal usage fail to reveal survival
  differences, so the combined-modality design is required.'
- No fitted program communities or usage data exist yet (e.g., raw counts before cNMF
  factorization and mosaicMPI network integration) — run program discovery and integration
  first.
- Clinical survival metadata (overall survival times and censoring indicators) is
  unavailable for the cohort, making Kaplan-Meier/log-rank testing impossible.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: mosaicMPI
  role: Provides the integrated network (network_integration.pkl.gz) whose per-patient
    program-community usage (classical C1/CLA, mesenchymal MES) is extracted across
    the RNA and protein datasets to define the survival groups.
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Loads the pickled network integration and merges stratification labels with
    CPTAC clinical survival metadata (overall survival times and censoring indicators).
- name: NumPy
  role: Numeric handling of per-patient community-usage arrays (and program correlation
    matrices compatible with NumPy) when thresholding usage into the co-usage versus
    classical-only groups.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:9a3307fce0e11a2877b54a7e56bd569b178c43fda0c830303c3b536102dc4096
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# community-usage-patient-stratification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratifies CPTAC GBM patients by cross-modality co-usage of mosaicMPI program communities — classical (CLA/C1) and mesenchymal (MES) usage high in both RNA and protein modalities — and compares Kaplan-Meier survival against a CLA-only group via log-rank test (reported p = 0.0123, Figure 4E).

## When to use

Use when per-patient program-community usage (e.g., classical C1/CLA and mesenchymal MES) from an integrated mosaicMPI RNA+protein landscape must define survival groups tested by Kaplan-Meier/log-rank against CPTAC clinical metadata.

## When NOT to use

- Single-modality analysis is intended: RNA-only or protein-only stratifications of CPTAC GBM patients by classical and mesenchymal usage fail to reveal survival differences, so the combined-modality design is required.
- No fitted program communities or usage data exist yet (e.g., raw counts before cNMF factorization and mosaicMPI network integration) — run program discovery and integration first.
- Clinical survival metadata (overall survival times and censoring indicators) is unavailable for the cohort, making Kaplan-Meier/log-rank testing impossible.

## Inputs

- mosaicMPI network integration pickle (network_integration.pkl.gz) containing all datasets, correlations, metadata, and program usage
- Per-patient community usage values for the classical (C1) and mesenchymal program communities in both the RNA and protein datasets
- CPTAC clinical survival metadata: overall survival times and censoring indicators per patient

## Outputs

- kaplan_meier_figure.png — Kaplan-Meier survival curves comparing CLA+MES co-usage versus classical-only patients
- logrank_test — log-rank statistic and p-value (expected ≈ 0.0123)
- Patient stratification labels (co-usage vs classical-only) merged with clinical survival metadata

## How to apply

Load the mosaicMPI network integration pickle (network_integration.pkl.gz, produced in task_004), which bundles all datasets, program correlations, metadata, and program usage. Extract per-patient community usage values for the classical (C1) and mesenchymal program communities separately from the RNA and protein datasets. Stratify into a co-usage group (high usage of both classical and mesenchymal programs in both modalities) versus a classical-only group — the key decision point, because single-modality RNA-only or protein-only stratifications fail to reveal survival differences. Merge group labels with the CPTAC clinical survival metadata (overall survival times and censoring indicators) using pandas. Fit Kaplan-Meier curves per group and compute the log-rank test comparing hazards; the expected result is a significant p ≈ 0.0123 with worse survival in the co-usage group (Figure 4E). Save the plot as kaplan_meier_figure.png and the log-rank statistic and p-value as logrank_test.

## Related tools

- **mosaicMPI** (Provides the integrated network (network_integration.pkl.gz) whose per-patient program-community usage (classical C1/CLA, mesenchymal MES) is extracted across the RNA and protein datasets to define the survival groups.) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Loads the pickled network integration and merges stratification labels with CPTAC clinical survival metadata (overall survival times and censoring indicators).)
- **NumPy** (Numeric handling of per-patient community-usage arrays (and program correlation matrices compatible with NumPy) when thresholding usage into the co-usage versus classical-only groups.)

## Examples

```
python -c "import pandas as pd; net = pd.read_pickle('network_integration.pkl.gz'); print(net)"
```

## Evaluation signals

- Log-rank p-value from the co-usage versus classical-only hazard comparison approximates the reported 0.0123 (Figure 4E).
- Both output files exist with correct formats: kaplan_meier_figure.png (Kaplan-Meier plot) and logrank_test (statistic plus p-value).
- RNA-only and protein-only stratifications of the same patients show no significant survival differences, confirming the cross-modality requirement.
- Every stratified patient maps to CPTAC survival metadata (overall survival time and censoring indicator) after the pandas merge; no unmatched or dropped samples.
- Directionality check: the co-usage group shows worse survival than the classical-only group in the Kaplan-Meier curves.

## Limitations

- Single-modality (RNA-only or protein-only) stratifications by classical and mesenchymal program usage fail to reveal survival differences; the signal only emerges when usage is maximized across both RNA and protein modalities.
- Potential loss of relevant information between modalities in mosaic integration because not all features are shared (article's own caveat).
- Depends on prior mosaicMPI integration output (network_integration.pkl.gz with program usage and clinical annotations propagated onto the program network); not applicable to raw, unintegrated expression data.

## Evidence

- [results] CLA+MES co-usage predicts worse survival: [evidence span withheld — anchor status: not_found]
- [methods] Per-patient usage extraction across modalities: [evidence span withheld — anchor status: not_found]
- [methods] KM curves and log-rank hazard comparison: [evidence span withheld — anchor status: not_found]
- [readme] Program communities interpreted with sample metadata: [evidence span withheld — anchor status: external_doc]
