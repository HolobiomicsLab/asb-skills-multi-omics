---
name: immune-cell-infiltration-estimation
description: Use when a cohort has computed iMLGAM risk scores (with 'riskscore' and
  'res' columns per sample) and the goal is to contrast tumor microenvironment immune
  metrics — leukocyte, lymphocyte, TIL, CD8+ T cell, fibroblast, Danaher scores —
  between low- and high-score groups.
when_to_use_negative:
- The cohort lacks computed iMLGAM risk scores — the stratification depends on the
  'riskscore' column from predict(ga_fit, ...)
- The goal is ICB response prediction or model training (base learner construction,
  ABESS feature selection, GA optimization) rather than post-hoc immune microenvironment
  comparison
- Only bulk gene-pair feature data is available without transcriptomic data needed
  for immune signature, cytolytic activity, and chemokine computations
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: iMLGAM
  role: R package providing GA_train()/predict(ga_fit, ...) to compute iMLGAM risk
    scores used for high/low stratification of the TCGA cohort
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Runtime environment in which iMLGAM scoring, immune metric estimation, and
    group-contrast statistics are computed
  repo: https://github.com/Yelab1994/iMLGAM
- name: qs
  role: Used to load the sample .qs cohort files (e.g., qread('Sample File.qs')) that
    supply the per-sample outcome and risk-score inputs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:199319c4bdb37395812bbafe090d84c7f3ff6799b35070b7b2eb64a6a7b6ccda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - iMLGAM
  - qs
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# immune-cell-infiltration-estimation

## Summary

Stratifies a cohort (e.g., TCGA) into high- and low-iMLGAM-score groups using the GA-trained model's 'riskscore' column, then contrasts per-sample tumor immune microenvironment metrics — leukocyte/lymphocyte/TIL fractions, CD8+ T cell abundance, Danaher gene-expression infiltration scores, fibroblast abundance, cytolytic activity, and chemokine expression — between groups to test whether low-score tumors are more immune-infiltrated.

## When to use

Use when a cohort has computed iMLGAM risk scores (with 'riskscore' and 'res' columns per sample) and the goal is to contrast tumor microenvironment immune metrics — leukocyte, lymphocyte, TIL, CD8+ T cell, fibroblast, Danaher scores — between low- and high-score groups.

## When NOT to use

- The cohort lacks computed iMLGAM risk scores — the stratification depends on the 'riskscore' column from predict(ga_fit, ...)
- The goal is ICB response prediction or model training (base learner construction, ABESS feature selection, GA optimization) rather than post-hoc immune microenvironment comparison
- Only bulk gene-pair feature data is available without transcriptomic data needed for immune signature, cytolytic activity, and chemokine computations

## Inputs

- Sample-level .qs data (e.g., 'Sample File.qs', 'ICI cohort gene pairs.qs') with 'res' outcome and 'riskscore' columns
- Pre-trained GA model object (ga_fit) from GA_train()
- TCGA pan-cancer transcriptomic dataset (Table S2 list of pan-cancer datasets)
- Per-sample immune metric data: leukocyte/lymphocyte/TIL fractions, CD8+ T cell abundance, Danaher gene-expression infiltration scores, fibroblast abundance

## Outputs

- A single results table with one row per immune metric and columns for group comparison statistics (high vs. low iMLGAM score contrasts)
- Direction-of-effect confirmation that low iMLGAM score specimens show enhanced CD8+ T cell infiltration and elevated immunogenicity

## How to apply

First compute iMLGAM scores per sample with the pre-trained GA model, e.g., riskscore <- predict(ga_fit, ...), keeping the outcome ('res') column aligned per sample as in the iMLGAM workflow. Split the cohort into high- and low-score groups by the 'riskscore' column using the training-cohort formula and cutoff. Estimate immune microenvironment quantities per sample: leukocyte, lymphocyte, and TIL fractions, CD8+ T cell abundance, Danaher gene-expression infiltration scores, and fibroblast abundance. Compute immune signature scores, cytolytic activity, and chemokine expression levels from the transcriptomic data per sample. Contrast each immune metric between the high- and low-score groups and assemble all group differences into a single results table with one row per immune metric and columns for the group comparison statistics. The expected direction of effect is that reduced iMLGAM scores correspond to an inflammatory profile with enhanced CD8+ T lymphocyte infiltration and elevated immunogenicity.

## Related tools

- **iMLGAM** (R package providing GA_train()/predict(ga_fit, ...) to compute iMLGAM risk scores used for high/low stratification of the TCGA cohort) — https://github.com/Yelab1994/iMLGAM
- **R** (Runtime environment in which iMLGAM scoring, immune metric estimation, and group-contrast statistics are computed) — https://github.com/Yelab1994/iMLGAM
- **qs** (Used to load the sample .qs cohort files (e.g., qread('Sample File.qs')) that supply the per-sample outcome and risk-score inputs)

## Examples

```
library(qs); dat <- qread("Sample File.qs"); trainres$riskscore <- predict(ga_fit, trainres)
```

## Evaluation signals

- Output file exists with one row per immune metric and columns for the group comparison statistics
- Direction of effect matches the report: larger fractions of leukocytes, lymphocytes, and TILs in the low iMLGAM score group
- Higher proportion of CD8+ T cells recovered in the low-score group, consistent with enhanced CD8+ T lymphocyte infiltration
- 'res' outcome column remains aligned per sample after splitting into high- and low-score groups

## Limitations

- The 11 RNA-seq cohorts analyzed represent a limited range of six cancer types, so pan-cancer generalizability requires further validation
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses
- Validation depends on using the training-cohort formula and cutoff; a different cutoff would change group membership and contrasts

## Evidence

- [abstract] tumors with low iMLGAM scores display distinctive immune microenvironment characteristics, including increased immune cell infiltration: "tumors with low iMLGAM scores display distinctive immune microenvironment characteristics, including increased immune cell infiltration and amplified antitumor immune responses"
- [discussion] specimens with reduced iMLGAM scores exhibited an inflammatory immunological profile with enhanced CD8+T lymphocyte infiltration and elevated immunogenicity: "specimens with reduced iMLGAM scores exhibited an inflammatory immunological profile, marked by enhanced CD8+T lymphocyte infiltration and elevated immunogenicity"
- [discussion] The comprehensive TCGA database was examined to characterize immunotherapeutic response patterns across malignancies.: "The comprehensive TCGA database was examined to characterize immunotherapeutic response patterns across malignancies."
- [readme] README risk score prediction call: "#Obtaining the evaluation score of the training set
trainres $riskscore <- predict(ga_fit ,trainres)"
