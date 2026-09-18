---
name: log-rank-hazard-comparison
description: Use when per-patient program usage (e.g. CLA/MES communities) is available
  in both RNA and protein modalities, clinical survival metadata exists, and single-modality
  stratifications fail to separate hazards.
when_to_use_negative:
- Patients lack clinical survival metadata (overall survival times/censoring indicators),
  so no hazard comparison can be fit
- Program usage exists in only one modality (RNA-only or protein-only) and a single-modality
  stratification is the intended comparison rather than cross-modality co-usage
- The analysis goal is program discovery or integration itself rather than downstream
  survival validation — use the mosaicMPI factorization/integration workflow instead
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: mosaicMPI
  role: Provides the integrated network and per-patient program/community usage across
    RNA and protein datasets that feed the stratification
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Holds per-patient usage matrices and clinical survival metadata for merging
    stratification labels with outcomes
- name: NumPy
  role: Handles numeric arrays of program usage and correlation outputs underlying
    the integration object
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:78e850376a0cdcbc471a44c8fb40b5031fbf42323a3b52493a070bc6580cb486
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.2
n_claims: 5
---

# log-rank-hazard-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Survival analysis skill that stratifies CPTAC GBM patients by cross-modality co-usage of classical (CLA) and mesenchymal (MES) program communities from the mosaicMPI integration, fits Kaplan-Meier curves, and tests hazard differences with a log-rank test (target p ≈ 0.0123, Figure 4E).

## When to use

Use when per-patient program usage (e.g. CLA/MES communities) is available in both RNA and protein modalities, clinical survival metadata exists, and single-modality stratifications fail to separate hazards.

## When NOT to use

- Patients lack clinical survival metadata (overall survival times/censoring indicators), so no hazard comparison can be fit
- Program usage exists in only one modality (RNA-only or protein-only) and a single-modality stratification is the intended comparison rather than cross-modality co-usage
- The analysis goal is program discovery or integration itself rather than downstream survival validation — use the mosaicMPI factorization/integration workflow instead

## Inputs

- network_integration.pkl.gz from the mosaicMPI workflow (datasets, correlations, metadata, and per-patient program/community usage)
- Per-patient classical (C1) and mesenchymal community usage values in both RNA and protein datasets
- CPTAC clinical survival metadata (overall survival times and censoring indicators)

## Outputs

- kaplan_meier_figure.png (Kaplan-Meier survival curves for co-usage vs classical-only groups)
- logrank_test (log-rank statistic and p-value, expected ≈ 0.0123)

## How to apply

Load the network integration produced by the mosaicMPI workflow (network_integration.pkl.gz) and extract per-patient community usage values for the classical (C1) and mesenchymal program communities in both the RNA and protein datasets. Define a co-usage group (high usage of both classical and mesenchymal programs across both modalities) versus a classical-only group, since the motivating question is whether single-modality (RNA-only or protein-only) stratifications mask the survival signal. Merge these labels with CPTAC overall survival times and censoring indicators, fit Kaplan-Meier curves per group, and compute the log-rank statistic comparing hazards. Save the curves as kaplan_meier_figure.png and the log-rank statistic/p-value as logrank_test; the co-usage group should show significantly worse survival than the CLA-only group.

## Related tools

- **mosaicMPI** (Provides the integrated network and per-patient program/community usage across RNA and protein datasets that feed the stratification) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Holds per-patient usage matrices and clinical survival metadata for merging stratification labels with outcomes)
- **NumPy** (Handles numeric arrays of program usage and correlation outputs underlying the integration object)

## Examples

```
import gzip, pickle; ni = pickle.load(gzip.open('network_integration.pkl.gz', 'rb')); usage = ni['program_usage']  # per-patient CLA/MES usage for RNA + protein, then fit KM and logrank_test
```

## Evaluation signals

- Log-rank p-value from the co-usage vs classical-only hazard comparison is approximately 0.0123, matching Figure 4E
- Both kaplan_meier_figure.png and logrank_test output files exist and have the correct formats
- Kaplan-Meier plot shows the CLA/MES co-usage group with worse survival than the CLA-only group
- Stratification is based on high usage of both classical and mesenchymal programs in both RNA and protein modalities, not a single modality
- Survival labels match the CPTAC clinical metadata (times plus censoring indicators) after merging

## Limitations

- Mosaic integration can suffer loss of relevant information between modalities because not all features are shared between datasets
- No software version information is stated for the tools/methods, complicating reproducibility
- The reported p-value depends on the specific stratification of co-usage vs classical-only groups; different thresholds may not reproduce p ≈ 0.0123

## Evidence

- [results] cross-modality co-usage survival finding: [evidence span withheld — anchor status: not_found]
