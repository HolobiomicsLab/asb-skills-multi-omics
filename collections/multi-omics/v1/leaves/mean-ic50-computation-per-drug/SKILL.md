---
name: mean-ic50-computation-per-drug
description: 'Use when reproducing CanSeer''s GDSC2 drug-sensitivity screening: you
  have per-cell-line, per-drug IC50 measurements, an LUSC cell-line annotation, and
  a list of candidate drugs, and need outlier-filtered mean IC50 per drug.'
when_to_use_negative:
- You need patient-specific therapy-response quantification (drug scores, efficacy,
  cytotoxic effect) rather than cell-line-averaged drug sensitivity aggregates
- The dataset is not GDSC2 IC50 measurements restricted to LUSC cell lines (e.g. it
  is CCLE, PRISM, or a different tumour type), so the outlier-removal and reference-value
  checks do not apply
tools:
- name: CanSeer
  role: Therapeutic screening workflow within which the GDSC2 IC50 filtering, outlier
    removal, and per-drug mean computation are embedded
  repo: github.com/BIRL/CanSeer
- name: pandas
  role: Used to compute the mean IC50 per drug across retained LUSC cell lines after
    outlier removal
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:e93e82d9600e796dab9e4c4e3bde6a9abf507082361542c2fad709864a4cfeae
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BIRL/CanSeer
  repo_url: https://github.com/BIRL/CanSeer
  tools:
  - CanSeer
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mean-ic50-computation-per-drug

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes average IC50 values per candidate drug from the GDSC2 drug sensitivity dataset after filtering to Lung NSCLC squamous cell carcinoma (LUSC) cell lines and removing outlier measurements, as performed during CanSeer's therapeutic screening.

## When to use

Use when reproducing CanSeer's GDSC2 drug-sensitivity screening: you have per-cell-line, per-drug IC50 measurements, an LUSC cell-line annotation, and a list of candidate drugs, and need outlier-filtered mean IC50 per drug.

## When NOT to use

- You need patient-specific therapy-response quantification (drug scores, efficacy, cytotoxic effect) rather than cell-line-averaged drug sensitivity aggregates
- The dataset is not GDSC2 IC50 measurements restricted to LUSC cell lines (e.g. it is CCLE, PRISM, or a different tumour type), so the outlier-removal and reference-value checks do not apply

## Inputs

- GDSC2 drug sensitivity dataset (IC50 measurements per cell line and drug) from the public GDSC2 release
- Cell-line annotations filtered to Lung NSCLC squamous cell carcinoma (LUSC) lines
- List of 18 candidate drugs: afatinib, afuresertib, APR-246, carmustine, dabrafenib, dinaciclib, ibrutinib, ipatasertib, nutlin-3a, olaparib, osimertinib, oxaliplatin, palbociclib, ribociclib, selumetinib, trametinib, ulixertinib, KU-55933

## Outputs

- Per-drug mean IC50 table for the 18 candidate drugs after outlier removal, e.g. mean_ic50_lusc_gdsc2.csv
- Ordered average IC50 values (e.g. 5.06 for afatinib, 0.08 for dinaciclib, 4.42 for osimertinib, 1.23 for trametinib)

## How to apply

Retrieve the GDSC2 drug sensitivity dataset containing IC50 measurements per cell line and drug from the public GDSC2 release, and filter the cell-line annotations to Lung NSCLC squamous cell carcinoma lines. Subset the screening records to the 18 candidate drugs (afatinib, afuresertib, APR-246, carmustine, dabrafenib, dinaciclib, ibrutinib, ipatasertib, nutlin-3a, olaparib, osimertinib, oxaliplatin, palbociclib, ribociclib, selumetinib, trametinib, ulixertinib, KU-55933). Remove outlier IC50 measurements within each drug-cell-line group before averaging, so that extreme sensitivity/resistance readings do not skew the mean. Compute the mean IC50 per drug across the retained LUSC cell lines using pandas and write the per-drug mean table to mean_ic50_lusc_gdsc2.csv. Validate by comparing computed means against the reported reference values (dinaciclib 0.08, trametinib 1.23, osimertinib 4.42, afatinib 5.06), which reflect the expected ordering of effective repositioned drugs.

## Related tools

- **CanSeer** (Therapeutic screening workflow within which the GDSC2 IC50 filtering, outlier removal, and per-drug mean computation are embedded) — github.com/BIRL/CanSeer
- **pandas** (Used to compute the mean IC50 per drug across retained LUSC cell lines after outlier removal)

## Examples

```
df.groupby('drug_name')['IC50'].mean().to_csv('mean_ic50_lusc_gdsc2.csv')
```

## Evaluation signals

- Computed mean IC50 for anchor drugs matches the reported reference values: dinaciclib 0.08, trametinib 1.23, osimertinib 4.42, afatinib 5.06
- Output table contains exactly 18 drugs, one mean IC50 each, matching the reported value set (5.06, 17.52, 707.84, 748.64, 203.12, 0.08, 113.09, 74.53, 265.12, 98.49, 4.42, 60.17, 65.97, 63.55, 28.35, 1.23, 18.80, 208.66)
- Outlier IC50 measurements were removed within each drug-cell-line group before averaging (means computed on retained records only)
- Only Lung NSCLC squamous cell carcinoma cell lines were retained in the aggregation
- The per-drug mean IC50 table is written to mean_ic50_lusc_gdsc2.csv with one row per drug

## Limitations

- Mean IC50 values are cell-line-averaged aggregates and do not capture patient-to-patient variability of therapeutic response, which CanSeer addresses via patient-specific drug scores and dynamical network analysis
- Results are specific to the GDSC2 release and the LUSC cell-line subset; other datasets or tumour types will yield different means
- Reported means depend on the outlier-removal procedure within each drug-cell-line group; deviations from that procedure will change the averages

## Evidence

- [other] Remove outlier IC50 measurements within each drug-cell-line group.: [evidence span withheld — anchor status: not_found]
- [abstract] CanSeer provides an all-in-one approach that ascertains actionable targets, repositions FDA-approved drugs, furnishes quantitative cues on therapy responses: [evidence span withheld — anchor status: not_found]
- [other] Subset the screening records to the 18 candidate drugs: afatinib, afuresertib, APR-246, carmustine, dabrafenib, dinaciclib, ibrutinib, ipatasertib, nutlin-3a, olaparib, osimertinib: [evidence span withheld — anchor status: not_found]
