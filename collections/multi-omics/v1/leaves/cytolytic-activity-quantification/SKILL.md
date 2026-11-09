---
name: cytolytic-activity-quantification
description: Use when a bulk RNA-seq cohort (e.g., TCGA) has been stratified by a
  score such as iMLGAM riskscore and you need per-sample cytolytic activity alongside
  other immune metrics (leukocyte/lymphocyte/TIL fractions, CD8+ T cell abundance)
  to compare groups.
when_to_use_negative:
- Input has no gene-level expression matrix (e.g., only clinical annotations or precomputed
  scores), so GZMA/PRF1-based CYT cannot be computed
- Data are single-cell; per-cell cytotoxic effector expression can be read directly
  and per-sample CYT aggregation is unnecessary
- The goal is ICB response prediction itself (use the iMLGAM scoring workflow, not
  this immune-comparison skill)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: iMLGAM
  role: Supplies the riskscore used to stratify samples into groups whose cytolytic
    activity is compared; installed from GitHub into R
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Environment in which sample .qs data are loaded (qs package) and per-sample
    cytolytic/immune metrics are computed from the transcriptomic matrix
- name: qs
  role: Loads the sample .qs files (e.g., 'Sample File.qs', 'ICI cohort gene pairs.qs')
    containing the cohort expression and outcome data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:7741ca2f003c321c2a3d50c5fb61614b8156d83d4aac34f4b43d6e65d7aed7c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - iMLGAM
  - qs
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Cytolytic activity quantification

## Summary

Computes per-sample cytolytic activity (CYT) from bulk transcriptomic data (e.g., TCGA RNA-seq) as a quantitative readout of effector T-cell/granzyme-mediated antitumor immunity, then contrasts it between biomarker-defined groups such as low- vs high-iMLGAM-score cohorts.

## When to use

Use when a bulk RNA-seq cohort (e.g., TCGA) has been stratified by a score such as iMLGAM riskscore and you need per-sample cytolytic activity alongside other immune metrics (leukocyte/lymphocyte/TIL fractions, CD8+ T cell abundance) to compare groups.

## When NOT to use

- Input has no gene-level expression matrix (e.g., only clinical annotations or precomputed scores), so GZMA/PRF1-based CYT cannot be computed
- Data are single-cell; per-cell cytotoxic effector expression can be read directly and per-sample CYT aggregation is unnecessary
- The goal is ICB response prediction itself (use the iMLGAM scoring workflow, not this immune-comparison skill)

## Inputs

- TCGA bulk RNA-seq expression matrix (sample × gene) containing GZMA and PRF1
- Per-sample iMLGAM risk scores (riskscore column) and outcome labels (res column)
- High/low iMLGAM score group assignments derived from the training-cohort formula and cutoff

## Outputs

- Per-sample cytolytic activity (CYT) values
- Group-contrast statistics table with one row per immune metric (CYT, leukocyte/lymphocyte/TIL fractions, CD8+ T cell abundance, chemokine levels) comparing high- vs low-iMLGAM groups

## How to apply

After computing iMLGAM risk scores via predict(ga_fit, ...) and splitting samples into high- and low-score groups using the 'riskscore' column with the 'res' outcome kept aligned, derive cytolytic activity per sample from the TCGA transcriptomic expression matrix together with immune signature scores and chemokine expression levels. Standard practice expresses CYT as the geometric mean of granzyme A (GZMA) and perforin 1 (PRF1) expression, yielding one non-negative value per specimen. Contrast CYT (and each companion immune metric) between the high- and low-score groups and assemble all group differences into a single results table with one row per immune metric and group-comparison statistics. The expected direction of effect is that low-iMLGAM specimens show elevated cytolytic activity consistent with enhanced CD8+ T lymphocyte infiltration and elevated immunogenicity, which validates that the score indeed captures immune-inflammatory tumors.

## Related tools

- **iMLGAM** (Supplies the riskscore used to stratify samples into groups whose cytolytic activity is compared; installed from GitHub into R) — https://github.com/Yelab1994/iMLGAM
- **R** (Environment in which sample .qs data are loaded (qs package) and per-sample cytolytic/immune metrics are computed from the transcriptomic matrix)
- **qs** (Loads the sample .qs files (e.g., 'Sample File.qs', 'ICI cohort gene pairs.qs') containing the cohort expression and outcome data)

## Examples

```
library(qs); dat <- qread("Sample File.qs"); cyt <- sqrt(dat["GZMA", ] * dat["PRF1", ]); groups <- ifelse(riskscore < cutoff, "low", "high"); wilcox.test(cyt ~ groups)
```

## Evaluation signals

- Each sample yields exactly one non-negative CYT value computable from the geometric mean of GZMA and PRF1 expression
- Recovered group contrast matches the reported direction of effect: reduced iMLGAM scores correspond to elevated cytolytic activity/inflammatory profile with enhanced CD8+ T lymphocyte infiltration
- Output file exists with one row per immune metric and columns holding the group-comparison statistics
- Group assignment derived from the riskscore column keeps the res outcome column aligned per sample
- Sample counts after grouping match the input cohort with no samples dropped due to NA outcomes

## Limitations

- The analysis cohorts (11 RNA-seq cohorts) cover only six cancer types, so CYT-vs-score relationships may not generalize pan-cancer without prospective ICB trial validation
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses
- Associations between low score and high cytolytic activity are observational; mechanisms of immune evasion (e.g., CEP55–PD-L1 regulation) are not resolved by this metric

## Evidence

- [methods] workflow step: compute cytolytic activity from transcriptomic data: [evidence span withheld — anchor status: not_found]
- [discussion] low-score tumors show inflammatory, cytotoxic profile: "specimens with reduced iMLGAM scores exhibited an inflammatory immunological profile, marked by enhanced CD8+T lymphocyte infiltration and elevated immunogenicity"
- [abstract] low-score tumors have amplified antitumor immune responses: "tumors with low iMLGAM scores display distinctive immune microenvironment characteristics, including increased immune cell infiltration and amplified antitumor immune responses"
- [readme] loading .qs data in R before immune metrics: "dat <- qread("Sample File.qs")"
- [readme] risk scores used for grouping: "testres$riskscore<- predict(ga_fit ,testres)"
