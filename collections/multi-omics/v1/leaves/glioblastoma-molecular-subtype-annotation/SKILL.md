---
name: glioblastoma-molecular-subtype-annotation
description: Use when CPTAC GBM bulk RNA and protein data have been factorized into
  program communities (via cNMF/mosaicMPI) and patients must be labeled by classical
  (C1) vs mesenchymal program usage, or by cross-modality co-usage, to test survival
  associations such as in Figure 4E.
when_to_use_negative:
- The datasets have not been factorized into program communities and integrated across
  modalities (no mosaicMPI network integration exists) — subtype usage values cannot
  be extracted
- Only a single modality is available and the goal is single-modality subtype stratification
  — the reported survival difference requires combined RNA-plus-protein stratification
- The cohort lacks clinical survival metadata (overall survival times and censoring
  indicators), making Kaplan-Meier/log-rank analysis impossible
edam_topics:
- http://edamontology.org/topic_2640
tools:
- name: mosaicMPI
  role: Provides the integrated program network (network_integration.pkl.gz) from
    which per-patient classical and mesenchymal community usage is extracted in both
    RNA and protein modalities, and propagates clinical/molecular annotations onto
    the program network
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Holds per-patient usage tables and clinical metadata as data matrices for
    merging stratification labels with survival times and censoring indicators
- name: NumPy
  role: Numerical handling of the per-program correlation/usage arrays extracted from
    the integration (e.g. pearson.df.npz-compatible structures)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:3b5d8465649624014a39fe9f0de0bf62f9f5c088e048029c96028448a3161356
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# glioblastoma-molecular-subtype-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates CPTAC glioblastoma patients with classical (C1/CLA) and mesenchymal (MES) program usage derived from mosaicMPI-integrated RNA and protein modalities, and stratifies them into subtype-usage groups for survival analysis. It captures the finding that cross-modality co-usage of CLA and MES programs, rather than single-modality stratification, reveals a survival difference (log-rank p = 0.0123).

## When to use

Use when CPTAC GBM bulk RNA and protein data have been factorized into program communities (via cNMF/mosaicMPI) and patients must be labeled by classical (C1) vs mesenchymal program usage, or by cross-modality co-usage, to test survival associations such as in Figure 4E.

## When NOT to use

- The datasets have not been factorized into program communities and integrated across modalities (no mosaicMPI network integration exists) — subtype usage values cannot be extracted
- Only a single modality is available and the goal is single-modality subtype stratification — the reported survival difference requires combined RNA-plus-protein stratification
- The cohort lacks clinical survival metadata (overall survival times and censoring indicators), making Kaplan-Meier/log-rank analysis impossible

## Inputs

- network_integration.pkl.gz from task_004 (mosaicMPI network integration with datasets, correlations, metadata, and program usage)
- Per-patient community usage values for classical (C1/CLA) and mesenchymal (MES) program communities in RNA and protein datasets
- CPTAC clinical survival metadata (overall survival times and censoring indicators)

## Outputs

- Patient stratification labels: CLA+MES co-usage group vs CLA-only group
- kaplan_meier_figure.png (Kaplan-Meier survival plot for the two groups)
- logrank_test (log-rank statistic and p-value, expected ≈ 0.0123 per Figure 4E)

## How to apply

Load the network integration from task_004 (network_integration.pkl.gz), which contains all datasets, correlations, metadata, and program usage produced by mosaicMPI's community-level integration. Extract per-patient community usage values for the classical (C1) and mesenchymal program communities in both the RNA and protein datasets, exploiting mosaicMPI's ability to propagate clinical and molecular annotations onto the program network across modalities without shared sample IDs. Stratify patients into a co-usage group (high usage of both classical and mesenchymal programs in both modalities) versus a classical-only group. Merge these labels with the CPTAC clinical survival metadata (overall survival times and censoring indicators), then fit Kaplan-Meier curves per group and compute a log-rank test on the hazards. Save the Kaplan-Meier plot as kaplan_meier_figure.png and the log-rank statistic and p-value as logrank_test; a valid result reproduces the reported p ≈ 0.0123 (Figure 4E) for CLA+MES co-usage vs CLA-only.

## Related tools

- **mosaicMPI** (Provides the integrated program network (network_integration.pkl.gz) from which per-patient classical and mesenchymal community usage is extracted in both RNA and protein modalities, and propagates clinical/molecular annotations onto the program network) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Holds per-patient usage tables and clinical metadata as data matrices for merging stratification labels with survival times and censoring indicators)
- **NumPy** (Numerical handling of the per-program correlation/usage arrays extracted from the integration (e.g. pearson.df.npz-compatible structures))

## Examples

```
mosaicmpi txt-to-h5ad --data cptac_snRNA_subsampled.txt --metadata cptac_snRNA_subsampled.metadata.txt -o cptac_snrna.h5ad
```

## Evaluation signals

- The log-rank p-value for the co-usage vs classical-only hazard comparison is approximately 0.0123, matching the reported Figure 4E result
- Both output files exist with correct formats: kaplan_meier_figure.png (Kaplan-Meier plot) and logrank_test (statistic and p-value)
- The co-usage group is defined as high usage of both classical and mesenchymal programs in BOTH RNA and protein modalities, not single-modality usage
- Single-modality (RNA-only or protein-only) stratifications of CLA/MES usage should show no significant survival difference, confirming the necessity of the combined-modality approach
- All patients in the stratification have matched CPTAC clinical survival times and censoring indicators (no unmapped samples)

## Limitations

- Potential loss of relevant information between modalities in mosaic integration because not all features are shared
- mosaicMPI handles sparse data (zeros) but not missing values, so feature coverage gaps across RNA and protein datasets constrain the integration
- No software version information is stated for the tools/methods discussed, which may affect reproducibility of exact p-values

## Evidence

- [results] A stratification maximizing usage across both RNA and protein modalities revealed that patients with co-usage of classical (CLA) and mesenchymal (MES) programs have significantly worse survival than the CLA-only group (p = 0.0123; Figure 4E).: [evidence span withheld — anchor status: not_found]
