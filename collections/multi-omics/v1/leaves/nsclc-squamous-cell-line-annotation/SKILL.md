---
name: nsclc-squamous-cell-line-annotation
description: Use when a CanSeer therapeutic-screening analysis requires GDSC2 IC50
  values for candidate drugs restricted to Lung NSCLC squamous cell carcinoma cell
  lines, with per-drug means computed after outlier removal.
when_to_use_negative:
- The analysis targets a cancer type or subtype other than Lung NSCLC squamous cell
  carcinoma — the cell-line filter and reference IC50 values are LUSC-specific
- The goal is synergy scoring or drug-combination prediction rather than single-agent
  sensitivity quantification — use CanSeer's combination-identification step instead
- Only a single patient's response prediction (drug score / treatment efficacy) is
  needed from the network model, without GDSC2 population-level IC50 grounding
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: CanSeer
  role: Therapeutic-screening methodology whose candidate-drug list (repositioned
    FDA-approved drugs for LUSC) is queried against GDSC2 and whose LUSC use case
    provides the reference mean IC50 values
  repo: github.com/BIRL/CanSeer
- name: pandas
  role: Used to compute the mean IC50 per drug across retained LUSC cell lines and
    to write mean_ic50_lusc_gdsc2.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:7f2b318d77f92f749016e5e5e1bc169e392f779a3349481a6a8187d30a9735d9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BIRL/CanSeer
  tools:
  - CanSeer
  - pandas
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# nsclc-squamous-cell-line-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates CanSeer therapeutic screening inputs to Lung NSCLC squamous cell carcinoma (LUSC) cell lines and computes per-drug average IC50 values from the GDSC2 drug sensitivity dataset after within-group outlier removal, supporting repositioned-drug prioritization for LUSC.

## When to use

Use when a CanSeer therapeutic-screening analysis requires GDSC2 IC50 values for candidate drugs restricted to Lung NSCLC squamous cell carcinoma cell lines, with per-drug means computed after outlier removal.

## When NOT to use

- The analysis targets a cancer type or subtype other than Lung NSCLC squamous cell carcinoma — the cell-line filter and reference IC50 values are LUSC-specific
- The goal is synergy scoring or drug-combination prediction rather than single-agent sensitivity quantification — use CanSeer's combination-identification step instead
- Only a single patient's response prediction (drug score / treatment efficacy) is needed from the network model, without GDSC2 population-level IC50 grounding

## Inputs

- GDSC2 drug sensitivity dataset (IC50 per cell line and drug) from the public GDSC2 release
- Cell-line annotation table mapping GDSC cell-line identifiers to tissue/cancer subtypes (Lung NSCLC squamous cell carcinoma)
- List of 18 candidate drugs from CanSeer's therapeutic screening (afatinib, afuresertib, APR-246, carmustine, dabrafenib, dinaciclib, ibrutinib, ipatasertib, nutlin-3a, olaparib, osimertinib, oxaliplatin, palbociclib, ribociclib, selumetinib, trametinib, ulixertinib, KU-55933)

## Outputs

- Per-drug mean IC50 table for LUSC cell lines (mean_ic50_lusc_gdsc2.csv), one row per candidate drug after outlier removal

## How to apply

Retrieve the GDSC2 drug sensitivity dataset (IC50 measurements per cell line and drug) from the public GDSC2 release, then filter the cell-line annotations to Lung NSCLC squamous cell carcinoma lines. Subset the screening records to the 18 candidate drugs surfaced by CanSeer's LUSC analysis: afatinib, afuresertib, APR-246, carmustine, dabrafenib, dinaciclib, ibrutinib, ipatasertib, nutlin-3a, olaparib, osimertinib, oxaliplatin, palbociclib, ribociclib, selumetinib, trametinib, ulixertinib, and KU-55933. Remove outlier IC50 measurements within each drug-cell-line group before aggregation, since IC50 screens contain sporadic extreme values that would otherwise skew drug comparisons. Compute the mean IC50 per drug across the retained LUSC cell lines with pandas and write the per-drug table to mean_ic50_lusc_gdsc2.csv. Reference means for verification include dinaciclib 0.08, trametinib 1.23, osimertinib 4.42, and afatinib 5.06 (nM), which identify the strongest sensitivities among the panel.

## Related tools

- **CanSeer** (Therapeutic-screening methodology whose candidate-drug list (repositioned FDA-approved drugs for LUSC) is queried against GDSC2 and whose LUSC use case provides the reference mean IC50 values) — github.com/BIRL/CanSeer
- **pandas** (Used to compute the mean IC50 per drug across retained LUSC cell lines and to write mean_ic50_lusc_gdsc2.csv)

## Examples

```
import pandas as pd; df[df.cell_line.isin(lusc_lines)].groupby('drug')['ic50'].mean().to_csv('mean_ic50_lusc_gdsc2.csv')
```

## Evaluation signals

- Exactly 18 candidate drugs appear in the output table, matching the CanSeer candidate list
- Computed means match the reported reference values (dinaciclib 0.08, trametinib 1.23, osimertinib 4.42, afatinib 5.06) after outlier removal
- All retained cell lines are annotated as Lung NSCLC squamous cell carcinoma before aggregation
- Mean IC50 for the most sensitive drugs (dinaciclib, trametinib, osimertinib, afatinib) falls well below the most resistant ones (carmustine 748.64, dabrafenib 707.84), preserving the expected sensitivity ordering
- Output file mean_ic50_lusc_gdsc2.csv exists with one mean per drug and no missing values

## Limitations

- GDSC2 IC50 means reflect cell-line population sensitivity, not patient-specific response; CanSeer's in silico predicted patient-specific drug cocktail sensitivity requires further evaluation with tumor-on-chip models
- No wet lab validation of predicted efficacious drug combinations and repositioned drugs has been performed, so IC50-derived prioritization cannot alone establish clinical acceptability
- Outlier removal within drug-cell-line groups changes the computed means, so raw (unfiltered) GDSC2 aggregation will not reproduce the reported reference values

## Evidence
