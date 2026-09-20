---
name: ic50-outlier-removal
description: Use when computing robust per-drug average IC50 values from GDSC2 (or
  similar) drug sensitivity datasets for a specific cancer subtype's cell lines, and
  raw IC50 replicates contain outliers that would skew group means.
when_to_use_negative:
- The input is not per-cell-line drug sensitivity data (e.g. patient-level pharmacokinetics
  or dose-response curves from a primary screen)
- A different cancer type or non-LUSC cell-line panel is targeted without re-filtering
  cell-line annotations accordingly
- Drugs outside the candidate list (or absent from GDSC2) are queried — the mean table
  will lack those rows
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3325
tools:
- name: CanSeer
  role: Therapeutic-screening pipeline within which the GDSC2 IC50 outlier removal
    and per-drug mean computation are executed against LUSC cell lines for the 18
    candidate drugs
  repo: https://github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:c5790f7f35e219bc255a64bb4e4b04f7f46273bad8d2c4e170b0f8801549c726
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3325
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# ic50-outlier-removal

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Removes outlier IC50 measurements within each drug–cell-line group from GDSC2 drug sensitivity data and computes per-drug mean IC50 values across a filtered set of lung squamous cell carcinoma (LUSC) cell lines, as performed in CanSeer's therapeutic screening.

## When to use

Use when computing robust per-drug average IC50 values from GDSC2 (or similar) drug sensitivity datasets for a specific cancer subtype's cell lines, and raw IC50 replicates contain outliers that would skew group means.

## When NOT to use

- The input is not per-cell-line drug sensitivity data (e.g. patient-level pharmacokinetics or dose-response curves from a primary screen)
- A different cancer type or non-LUSC cell-line panel is targeted without re-filtering cell-line annotations accordingly
- Drugs outside the candidate list (or absent from GDSC2) are queried — the mean table will lack those rows

## Inputs

- GDSC2 drug sensitivity dataset (IC50 measurements per cell line and drug)
- GDSC cell-line annotation table with histology/subtype labels (e.g. Lung NSCLC squamous cell carcinoma)
- List of 18 candidate drugs to screen (afatinib, afuresertib, APR-246, carmustine, dabrafenib, dinaciclib, ibrutinib, ipatasertib, nutlin-3a, olaparib, osimertinib, oxaliplatin, palbociclib, ribociclib, selumetinib, trametinib, ulixertinib, KU-55933)

## Outputs

- Per-drug mean IC50 table across retained LUSC cell lines (e.g. mean_ic50_lusc_gdsc2.csv)
- Outlier-filtered IC50 measurements per drug–cell-line group

## How to apply

Retrieve the GDSC2 drug sensitivity dataset (IC50 per cell line and drug) from the public GDSC2 release, then filter cell-line annotations to the target subtype — here Lung NSCLC squamous cell carcinoma (LUSC) lines. Subset the records to the candidate drugs under screening (e.g. afatinib, afuresertib, APR-246, carmustine, dabrafenib, dinaciclib, ibrutinib, ipatasertib, nutlin-3a, olaparib, osimertinib, oxaliplatin, palbociclib, ribociclib, selumetinib, trametinib, ulixertinib, KU-55933). Before aggregating, remove outlier IC50 measurements within each drug–cell-line group so that extreme sensitivity/resistance values do not dominate the mean. Compute the mean IC50 per drug across retained LUSC cell lines using pandas and write the per-drug table (e.g. mean_ic50_lusc_gdsc2.csv). Rationale: outlier removal within groups yields stable quantitative therapy-response cues (efficacy/cytotoxicity) that CanSeer uses when ranking candidate therapeutics.

## Related tools

- **CanSeer** (Therapeutic-screening pipeline within which the GDSC2 IC50 outlier removal and per-drug mean computation are executed against LUSC cell lines for the 18 candidate drugs) — https://github.com/BIRL/CanSeer

## Examples

```
df = pd.read_csv('gdsc2_ic50.csv'); lusc = df[(df.histology=='Lung NSCLC')&(df.subtype=='squamous cell carcinoma')]; means = lusc.groupby('drug')['ic50'].apply(remove_outliers).groupby('drug').mean(); means.to_csv('mean_ic50_lusc_gdsc2.csv')
```

## Evaluation signals

- Sanity-check computed means against reference values: dinaciclib 0.08, trametinib 1.23, osimertinib 4.42, afatinib 5.06
- The full ordered mean-IC50 list should match: 5.06, 17.52, 707.84, 748.64, 203.12, 0.08, 113.09, 74.53, 265.12, 98.49, 4.42, 60.17, 65.97, 63.55, 28.35, 1.23, 18.80, 208.66 for the 18 candidate drugs respectively
- Output table contains exactly 18 rows (one per candidate drug) with mean IC50 values after outlier removal
- Each retained IC50 group contains no outlier measurements per the removal criterion applied within drug–cell-line groups
- Written CSV (mean_ic50_lusc_gdsc2.csv) parses with pandas and covers only Lung NSCLC squamous cell carcinoma cell lines

## Limitations

- The published model/screen may lack coverage for certain mutations or drugs in some patients/lines, potentially yielding inconclusive outcomes
- Outlier-removal criteria and cell-line filtering are specific to the CanSeer LUSC GDSC2 screening; results may not transfer directly to other subtypes or GDSC1 data
- In silico drug-response estimates require further experimental validation before clinical use

## Evidence

- [other] Average IC50 values computed after the removal of outliers for the candidate drugs were 5.06, 17.52, 707.84, 748.64, 203.12, 0.08, 113.09, 74.53, 265.12, 98.49, 4.42, 60.17, 65.97, 63.55, 28.35, 1.23, 18.80 and 208.66, respectively.: [evidence span withheld — anchor status: not_found]
- [other] Subset the screening records to the 18 candidate drugs: afatinib, afuresertib, APR-246, carmustine, dabrafenib, dinaciclib, ibrutinib, ipatasertib, nutlin-3a, olaparib, osimertinib, oxaliplatin, palbociclib, ribociclib, selumetinib, trametinib, ulixertinib, KU-55933.: [evidence span withheld — anchor status: not_found]
