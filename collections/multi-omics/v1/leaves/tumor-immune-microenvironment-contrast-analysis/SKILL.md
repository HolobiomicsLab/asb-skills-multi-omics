---
name: tumor-immune-microenvironment-contrast-analysis
description: Use when you have per-sample iMLGAM risk scores (riskscore column) with
  aligned outcome labels (res) for a cohort such as TCGA and need to test whether
  low-score tumors show enhanced immune infiltration and immunogenicity.
when_to_use_negative:
- The goal is to train or re-fit the iMLGAM model itself (base learner construction,
  ABESS feature selection, GA_train) rather than to contrast immune metrics in an
  already-scored cohort
- Input samples lack computed iMLGAM risk scores or a pre-trained ga_fit model, so
  no high/low score stratification can be made
- The cohort has no transcriptomic data, preventing estimation of gene-expression-based
  metrics such as Danaher infiltration scores, cytolytic activity, and chemokine levels
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: iMLGAM
  role: R package providing the GA-trained model (GA_train/ga_fit) and predict() used
    to compute the iMLGAM riskscore that defines the high/low strata being contrasted
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Language in which the score computation, cohort stratification, immune metric
    estimation, and group contrasts are performed
  repo: https://github.com/Yelab1994/iMLGAM
- name: qs
  role: Package used to load the sample .qs files (e.g. via qread) that supply the
    outcome and feature data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:3effe028cc655bbaab95ad1b5837cfb65b18d15a8a493a4744fdf0d21d12d80e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - iMLGAM
  - qs
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# tumor-immune-microenvironment-contrast-analysis

## Summary

Contrasts immune microenvironment metrics (leukocyte/lymphocyte/TIL fractions, CD8+ T cell abundance, Danaher scores, cytolytic activity, chemokines) between high- and low-iMLGAM-score tumor samples to show that low-score tumors are more immune-infiltrated and immunogenic.

## When to use

Use when you have per-sample iMLGAM risk scores (riskscore column) with aligned outcome labels (res) for a cohort such as TCGA and need to test whether low-score tumors show enhanced immune infiltration and immunogenicity.

## When NOT to use

- The goal is to train or re-fit the iMLGAM model itself (base learner construction, ABESS feature selection, GA_train) rather than to contrast immune metrics in an already-scored cohort
- Input samples lack computed iMLGAM risk scores or a pre-trained ga_fit model, so no high/low score stratification can be made
- The cohort has no transcriptomic data, preventing estimation of gene-expression-based metrics such as Danaher infiltration scores, cytolytic activity, and chemokine levels

## Inputs

- Sample .qs data file (e.g. 'Sample File.qs' or 'ICI cohort gene pairs.qs') with outcome ('res') first column and feature columns
- Pre-trained GA model object (ga_fit) from GA_train in the iMLGAM/iMLAGA R package
- TCGA pan-cancer transcriptomic dataset (Table S2 lists the pan-cancer transcriptomic datasets)
- Per-sample outcome/alignment table with 'riskscore' and 'res' columns

## Outputs

- Single results table with one row per immune metric (leukocytes, lymphocytes, TILs, CD8+ T cells, Danaher scores, fibroblasts, immune signatures, cytolytic activity, chemokines) and columns for high- vs low-score group comparison statistics
- Per-sample iMLGAM risk scores (riskscore column) with aligned outcome (res) labels

## How to apply

Load the sample .qs data with the qs package in R and compute iMLGAM risk scores via predict(ga_fit, ...), producing a 'riskscore' column while keeping the 'res' outcome column aligned per sample. Split the cohort into high- and low-score groups using the iMLGAM score from the training-cohort formula and cutoff. Per sample, estimate immune metrics: leukocyte/lymphocyte/TIL fractions, CD8+ T cell abundance, Danaher gene-expression infiltration scores, fibroblast abundance, immune signature scores, cytolytic activity, and chemokine expression from the transcriptomic data. Contrast each metric between the high- and low-score groups and assemble all group differences into a single results table with one row per immune metric and columns for group comparison statistics. Validate that recovered contrasts match the reported direction of effect: reduced iMLGAM scores correspond to an inflammatory profile with enhanced CD8+ T lymphocyte infiltration and elevated immunogenicity.

## Related tools

- **iMLGAM** (R package providing the GA-trained model (GA_train/ga_fit) and predict() used to compute the iMLGAM riskscore that defines the high/low strata being contrasted) — https://github.com/Yelab1994/iMLGAM
- **R** (Language in which the score computation, cohort stratification, immune metric estimation, and group contrasts are performed) — https://github.com/Yelab1994/iMLGAM
- **qs** (Package used to load the sample .qs files (e.g. via qread) that supply the outcome and feature data)

## Examples

```
trainres$riskscore <- predict(ga_fit, trainres)
```

## Evaluation signals

- Recovered contrasts match the reported direction of effect: specimens with reduced iMLGAM scores exhibit an inflammatory profile with enhanced CD8+ T lymphocyte infiltration and elevated immunogenicity
- Significantly larger fractions of leukocytes, lymphocytes, and TILs in the low iMLGAM score group, with a higher proportion of CD8+ T cells
- Output file exists with one row per immune metric and columns for the group comparison statistics
- 'riskscore' values are produced via predict(ga_fit, ...) and the 'res' outcome column remains aligned per sample after stratification

## Limitations

- The 11 RNA-seq cohorts cover only six cancer types; pan-cancer predictive efficacy of the iMLGAM score requires validation in prospective ICB trials
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses
- No large-scale, multi-center pan-cancer cohort validation, limiting the model's generalizability

## Evidence

- [abstract] tumors with low iMLGAM scores display distinctive immune microenvironment characteristics, including increased immune cell infiltration and amplified antitumor immune responses: "tumors with low iMLGAM scores display distinctive immune microenvironment characteristics, including increased immune cell infiltration and amplified antitumor immune responses"
- [discussion] specimens with reduced iMLGAM scores exhibited an inflammatory immunological profile with enhanced CD8+ T lymphocyte infiltration and elevated immunogenicity: "specimens with reduced iMLGAM scores exhibited an inflammatory immunological profile, marked by enhanced CD8+T lymphocyte infiltration and elevated immunogenicity"
- [readme] Load the sample .qs data with the qs package and score each set using the formula constructed in Step 3: "testres$riskscore<- predict(ga_fit ,testres)"
