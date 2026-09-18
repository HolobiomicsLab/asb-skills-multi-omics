---
name: cell-line-subsetting-by-cancer-type
description: Use when a drug-sensitivity or pharmacogenomic query must be restricted
  to cell lines of a specific cancer subtype (e.g., Lung NSCLC squamous cell carcinoma)
  before computing per-drug statistics like mean IC50.
when_to_use_negative:
- Analysis targets patient-specific personalized models (CanSeer's core use case uses
  patient genomic/RNA-seq data, not bulk cell-line panels)
- The research question requires pan-cancer or multi-subtype comparison rather than
  a single subtype like LUSC
- The goal is synergy-based drug-combination prioritization rather than single-drug
  sensitivity statistics
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0208
tools:
- name: CanSeer
  role: Therapeutic-screening pipeline whose drug-response quantification step queries
    GDSC2 against subtype-filtered cell lines to obtain cytotoxic-effect (IC50) cues
  repo: https://github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:293e749fbe0a1744795ad882f239640bd2f28cac8f70444808ef08457889a8a7
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0208
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 3
---

# Cell-line subsetting by cancer type

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Restricts a public drug-sensitivity dataset (GDSC2 IC50 measurements) to annotated cell lines of a specific cancer subtype — here Lung NSCLC squamous cell carcinoma (LUSC) — and then computes per-drug summary statistics such as mean IC50 after outlier removal, as done in CanSeer's therapeutic screening of 18 candidate drugs.

## When to use

Use when a drug-sensitivity or pharmacogenomic query must be restricted to cell lines of a specific cancer subtype (e.g., Lung NSCLC squamous cell carcinoma) before computing per-drug statistics like mean IC50.

## When NOT to use

- Analysis targets patient-specific personalized models (CanSeer's core use case uses patient genomic/RNA-seq data, not bulk cell-line panels)
- The research question requires pan-cancer or multi-subtype comparison rather than a single subtype like LUSC
- The goal is synergy-based drug-combination prioritization rather than single-drug sensitivity statistics

## Inputs

- GDSC2 drug sensitivity dataset (IC50 measurements per cell line and drug) from the public GDSC2 release
- Cell-line annotations with cancer-type/subtype labels (to filter Lung NSCLC squamous cell carcinoma lines)
- List of candidate drug names (e.g., 18 CanSeer screening candidates)

## Outputs

- Per-drug mean IC50 table for the retained LUSC cell lines (written to mean_ic50_lusc_gdsc2.csv)
- Ordered average IC50 values after outlier removal (e.g., 5.06, 17.52, 707.84, ..., 208.66 for the 18 candidate drugs)

## How to apply

Retrieve the GDSC2 drug sensitivity dataset (IC50 measurements per cell line and drug) from the public GDSC2 release, then filter the cell-line annotations to Lung NSCLC squamous cell carcinoma lines. Subset the screening records to the candidate drugs of interest (in CanSeer's screening: afatinib, afuresertib, APR-246, carmustine, dabrafenib, dinaciclib, ibrutinib, ipatasertib, nutlin-3a, olaparib, osimertinib, oxaliplatin, palbociclib, ribociclib, selumetinib, trametinib, ulixertinib, KU-55933). Remove outlier IC50 measurements within each drug-cell-line group before aggregation, since outliers can dominate the mean in small cell-line panels. Compute the mean IC50 per drug across the retained LUSC cell lines using pandas and write the per-drug table (e.g., mean_ic50_lusc_gdsc2.csv). The rationale is that subtype-restricted, outlier-cleaned means give the quantitative cytotoxicity cues (e.g., dinaciclib 0.08, trametinib 1.23, osimertinib 4.42, afatinib 5.06) used to rank repositioned drugs in CanSeer's therapeutic screening.

## Related tools

- **CanSeer** (Therapeutic-screening pipeline whose drug-response quantification step queries GDSC2 against subtype-filtered cell lines to obtain cytotoxic-effect (IC50) cues) — https://github.com/BIRL/CanSeer

## Examples

```
df = pd.read_csv('gdsc2_ic50.csv'); lusc = df[df['cell_line'].isin(lusc_lines) & df['drug'].isin(drugs)]; lusc.groupby('drug')['IC50'].mean().to_csv('mean_ic50_lusc_gdsc2.csv')
```

## Evaluation signals

- Computed mean IC50 for key drugs matches reported reference values: dinaciclib 0.08, trametinib 1.23, osimertinib 4.42, afatinib 5.06
- Exactly 18 per-drug means are produced, ordered to match the candidate drug list (5.06, 17.52, 707.84, 748.64, 203.12, 0.08, 113.09, 74.53, 265.12, 98.49, 4.42, 60.17, 65.97, 63.55, 28.35, 1.23, 18.80, 208.66)
- Cell-line table contains only Lung NSCLC squamous cell carcinoma lines after annotation filtering
- Outlier IC50 records were removed within each drug-cell-line group before averaging (means differ from naive groupby mean)
- Output CSV (mean_ic50_lusc_gdsc2.csv) is written with one row per drug and a mean-IC50 column

## Limitations

- Averages are specific to the GDSC2 release and the LUSC cell-line panel; they do not transfer directly to other cancer types or GDSC1 data
- Outlier-removal choices within drug-cell-line groups can shift the reported means; the removal rule must be documented for reproducibility
- Cell-line mean IC50 provides a population-level cytotoxicity cue and does not capture patient-specific drug scores that CanSeer computes from individual genomic/transcriptomic data

## Evidence

- [methods] Retrieve the GDSC2 drug sensitivity dataset (IC50 measurements per cell line and drug) from the public GDSC2 release. 2. Filter the cell-line annotations to Lung NSCLC squamous cell carcinoma lines.: [evidence span withheld — anchor status: not_found]
- [results] Average IC50 values computed after the removal of outliers for the candidate drugs were 5.06, 17.52, 707.84, 748.64, 203.12, 0.08, 113.09, 74.53, 265.12, 98.49, 4.42, 60.17, 65.97, 63.55, 28.35, 1.23, 18.80 and 208.66, respectively.: [evidence span withheld — anchor status: not_found]
