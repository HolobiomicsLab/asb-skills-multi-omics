---
name: risk-score-stratified-group-comparison
description: Use when a cohort has per-sample risk/predictive scores from a fitted
  model (e.g., iMLGAM scores computed via predict(ga_fit, ...)) and you need to test
  whether score-defined groups differ in immune infiltration or immunogenicity metrics.
when_to_use_negative:
- When no per-sample risk score exists yet — first compute scores via the iMLGAM GA
  model (Steps 1–4 of the README) before stratifying
- When the goal is to train or optimize the predictive signature itself (ABESS feature
  selection, base-learner construction, GA training) rather than to compare score-defined
  groups
- When cohort-level immune metrics cannot be estimated because the transcriptomic
  data lack the required gene coverage for Danaher scores, cytolytic activity, or
  chemokine panels
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: iMLGAM
  role: R package supplying GA_train and predict() to compute iMLGAM risk scores used
    to stratify the cohort into high- and low-score groups
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Runtime environment in which the iMLGAM package, qs data loading, and group-comparison
    statistics are executed
- name: qs
  role: Used to load the sample .qs data files (e.g., qread('Sample File.qs')) that
    carry the cohort features and outcomes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:fa8fae3e698b9c6ae81e93d590183b5c9965a3a6b59eaa092ff80c3146c5a39c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - iMLGAM
  - qs
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# risk-score-stratified-group-comparison

## Summary

Stratifies a patient cohort (e.g., TCGA) into high- and low-risk groups using a pre-trained model score such as the iMLGAM 'riskscore', then contrasts immune microenvironment metrics — leukocyte/lymphocyte/TIL fractions, CD8+ T cell abundance, Danaher infiltration scores, cytolytic activity, chemokine expression — between groups to characterize the tumor immune phenotype.

## When to use

Use when a cohort has per-sample risk/predictive scores from a fitted model (e.g., iMLGAM scores computed via predict(ga_fit, ...)) and you need to test whether score-defined groups differ in immune infiltration or immunogenicity metrics.

## When NOT to use

- When no per-sample risk score exists yet — first compute scores via the iMLGAM GA model (Steps 1–4 of the README) before stratifying
- When the goal is to train or optimize the predictive signature itself (ABESS feature selection, base-learner construction, GA training) rather than to compare score-defined groups
- When cohort-level immune metrics cannot be estimated because the transcriptomic data lack the required gene coverage for Danaher scores, cytolytic activity, or chemokine panels

## Inputs

- Sample .qs file (e.g., 'Sample File.qs') with trainset/validationset/testset entries, first column outcome ('res') and subsequent columns features
- Pre-trained GA model object (ga_fit from GA_train(trainres))
- TCGA cohort transcriptomic data (RNA-seq) with aligned clinical annotations
- Immune microenvironment annotation data (leukocyte/lymphocyte/TIL fractions, CD8+ T cell estimates, Danaher gene sets, fibroblast abundance)

## Outputs

- Per-sample iMLGAM risk scores ('riskscore' column) with aligned outcome labels
- High- and low-iMLGAM-score group assignments per sample
- A single results table with one row per immune metric and columns for the high-vs-low group comparison statistics

## How to apply

Load the sample .qs data with the qs package and the pre-trained GA model, then compute iMLGAM risk scores for the cohort via predict(ga_fit, ...), keeping the outcome ('res') column aligned per sample. Split samples into high- and low-score groups using the 'riskscore' column. Per sample, estimate immune microenvironment quantities: leukocyte/lymphocyte/TIL fractions, CD8+ T cell abundance, Danaher gene-expression infiltration scores, fibroblast abundance, immune signature scores, cytolytic activity, and chemokine expression from the transcriptomic data. Contrast each immune metric between the high- and low-score groups and assemble all group differences into a single results table with one row per immune metric and columns for the group comparison statistics. Validate by confirming the recovered contrasts match the reported direction of effect: reduced iMLGAM scores should correspond to an inflammatory profile with enhanced CD8+ T lymphocyte infiltration and elevated immunogenicity.

## Related tools

- **iMLGAM** (R package supplying GA_train and predict() to compute iMLGAM risk scores used to stratify the cohort into high- and low-score groups) — https://github.com/Yelab1994/iMLGAM
- **R** (Runtime environment in which the iMLGAM package, qs data loading, and group-comparison statistics are executed)
- **qs** (Used to load the sample .qs data files (e.g., qread('Sample File.qs')) that carry the cohort features and outcomes)

## Examples

```
library(qs); library(iMLGAM); scores <- predict(ga_fit, cohort); group <- ifelse(scores$riskscore > median(scores$riskscore), 'high', 'low')
```

## Evaluation signals

- Output file exists with exactly one row per immune metric and columns for the group comparison statistics
- Direction of effect matches the reported finding: significantly larger fractions of leukocytes, lymphocytes, and TILs, and a higher proportion of CD8+ T cells, in the low iMLGAM score group
- The 'riskscore' column is present and aligned sample-by-sample with the outcome ('res') column before group splitting
- Recovered contrasts reproduce an inflammatory, elevated-immunogenicity profile for reduced iMLGAM scores (higher cytolytic activity and chemokine expression in the low-score group)

## Limitations

- The 11 RNA-seq cohorts cover only six cancer types; pan-cancer predictive efficacy of the iMLGAM score requires further validation through prospective ICB trials
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses
- No large-scale, multi-center pan-cancer cohort validation, limiting the model's generalizability

## Evidence

- [discussion] specimens with reduced iMLGAM scores exhibited an inflammatory immunological profile, marked by enhanced CD8+T lymphocyte infiltration and elevated immunogenicity: "specimens with reduced iMLGAM scores exhibited an inflammatory immunological profile, marked by enhanced CD8+T lymphocyte infiltration and elevated immunogenicity"
- [abstract] tumors with low iMLGAM scores display distinctive immune microenvironment characteristics, including increased immune cell infiltration and amplified antitumor immune responses: "tumors with low iMLGAM scores display distinctive immune microenvironment characteristics, including increased immune cell infiltration and amplified antitumor immune responses"
- [readme] testres$riskscore<- predict(ga_fit ,testres): "testres$riskscore<- predict(ga_fit ,testres)"
- [discussion] The comprehensive TCGA database was examined to characterize immunotherapeutic response patterns across malignancies.: "The comprehensive TCGA database was examined to characterize immunotherapeutic response patterns across malignancies."
