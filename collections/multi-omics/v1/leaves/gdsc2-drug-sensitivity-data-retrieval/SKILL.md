---
name: gdsc2-drug-sensitivity-data-retrieval
description: Use when quantitative drug-response baselines (mean IC50 per drug) are
  needed for GDSC2 screening against LUSC cell lines, e.g. to support CanSeer's drug
  repositioning and therapeutic efficacy ranking.
when_to_use_negative:
- Input drug-response data is not GDSC2 IC50 measurements (e.g. AUC or CCLE pharmacological
  screens), where IC50-based means and outlier removal are not comparable.
- The aim is patient-specific drug-score computation or dynamical network analysis
  in CanSeer rather than population-level GDSC2 sensitivity baselines.
- Cell lines of a different histology are required; filtering is specific to Lung
  NSCLC squamous cell carcinoma lines.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: CanSeer
  role: Therapeutic screening pipeline whose candidate-drug list and LUSC use case
    define the GDSC2 query, filters and reported mean IC50 values
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:513c41428760c8b62410fa2b1e79f61fa320318816af5a7d6e848dd66eac9976
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# gdsc2-drug-sensitivity-data-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves the public GDSC2 drug sensitivity release (per-cell-line IC50 measurements), filters to Lung NSCLC squamous cell carcinoma cell lines, subsets to CanSeer's 18 candidate drugs, and computes outlier-removed mean IC50 values per drug for use in therapeutic screening.

## When to use

Use when quantitative drug-response baselines (mean IC50 per drug) are needed for GDSC2 screening against LUSC cell lines, e.g. to support CanSeer's drug repositioning and therapeutic efficacy ranking.

## When NOT to use

- Input drug-response data is not GDSC2 IC50 measurements (e.g. AUC or CCLE pharmacological screens), where IC50-based means and outlier removal are not comparable.
- The aim is patient-specific drug-score computation or dynamical network analysis in CanSeer rather than population-level GDSC2 sensitivity baselines.
- Cell lines of a different histology are required; filtering is specific to Lung NSCLC squamous cell carcinoma lines.

## Inputs

- GDSC2 drug sensitivity dataset (IC50 measurements per cell line and drug) from the public GDSC2 release
- GDSC cell-line annotation table with histology/subtype labels (to select Lung NSCLC squamous cell carcinoma lines)
- List of 18 candidate drugs (afatinib, afuresertib, APR-246, carmustine, dabrafenib, dinaciclib, ibrutinib, ipatasertib, nutlin-3a, olaparib, osimertinib, oxaliplatin, palbociclib, ribociclib, selumetinib, trametinib, ulixertinib, KU-55933)

## Outputs

- Per-drug mean IC50 table after outlier removal (mean_ic50_lusc_gdsc2.csv)
- Mean IC50 values for the 18 candidate drugs across LUSC cell lines (e.g. dinaciclib 0.08, trametinib 1.23, osimertinib 4.42, afatinib 5.06)

## How to apply

Retrieve the GDSC2 drug sensitivity dataset containing IC50 measurements per cell line and drug from the public GDSC2 release, then use the cell-line annotations to keep only Lung NSCLC squamous cell carcinoma (LUSC) lines. Subset the screening records to the 18 CanSeer candidate drugs: afatinib, afuresertib, APR-246, carmustine, dabrafenib, dinaciclib, ibrutinib, ipatasertib, nutlin-3a, olaparib, osimertinib, oxaliplatin, palbociclib, ribociclib, selumetinib, trametinib, ulixertinib and KU-55933. Within each drug–cell-line group, remove outlier IC50 measurements before aggregation to prevent single extreme sensitivities from skewing the drug-level estimate. Compute the mean IC50 per drug across the retained LUSC cell lines using pandas and export the per-drug table to mean_ic50_lusc_gdsc2.csv. This IC50 table provides the quantitative sensitivity context for CanSeer's repositioned-drug and combination findings.

## Related tools

- **CanSeer** (Therapeutic screening pipeline whose candidate-drug list and LUSC use case define the GDSC2 query, filters and reported mean IC50 values) — github.com/BIRL/CanSeer

## Examples

```
import pandas as pd; m = gdsc2[gdsc2.cell_line.isin(lusc_lines) & gdsc2.drug.isin(drugs)].groupby('drug')['IC50'].mean(); m.to_csv('mean_ic50_lusc_gdsc2.csv')
```

## Evaluation signals

- The computed mean IC50 for reference drugs matches reported values: dinaciclib 0.08, trametinib 1.23, osimertinib 4.42, afatinib 5.06.
- Exactly 18 candidate drugs appear in the output table, in the order 5.06, 17.52, 707.84, 748.64, 203.12, 0.08, 113.09, 74.53, 265.12, 98.49, 4.42, 60.17, 65.97, 63.55, 28.35, 1.23, 18.80, 208.66.
- Only Lung NSCLC squamous cell carcinoma cell lines remain after the histology filter (no adenocarcinoma or other lung lines present).
- Outlier removal was applied within each drug–cell-line group before averaging; recomputed means change if outliers are retained.
- Output file mean_ic50_lusc_gdsc2.csv exists and contains one mean IC50 row per drug.

## Limitations

- Mean IC50 values are population-level estimates over LUSC cell lines and do not capture patient-specific sensitivity, which CanSeer addresses separately via drug scores and network re-analysis.
- In silico predictions of sensitivity (including patient-specific drug cocktails) require further evaluation, e.g. with tumor-on-chip models, and lack wet-lab validation.
- The published model used for demonstration lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes, limiting downstream interpretation of screening results.

## Evidence

- [methods] Retrieve the GDSC2 drug sensitivity dataset (IC50 measurements per cell line and drug) from the public GDSC2 release. Filter the cell-line annotations to Lung NSCLC squamous cell carcinoma lines.: [evidence span withheld — anchor status: not_found]
