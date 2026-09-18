---
name: risk-score-validation-across-cohorts
description: Use when a continuous predictive score has been computed for immunotherapy
  cohorts and you must demonstrate it outperforms existing biomarkers (PD-L1, TMB)
  via ROC/AUC comparison across cohorts.
when_to_use_negative:
- The score has not yet been computed — run iMLGAM Steps 1–4 (abess feature selection,
  basic_learner, GA_train, predict) first to obtain riskscore values
- The outcome variable is continuous survival time rather than the binary immunotherapy
  response 'res' — ROC/AUC comparison is for binary response classification
- Only a single cohort is available and no competing biomarkers (PD-L1, TMB) can be
  measured, precluding the comparative validation this skill performs
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0203
tools:
- name: iMLGAM
  role: R package providing GA_train and predict to compute the riskscore that is
    benchmarked against PD-L1 and TMB
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Environment in which ROC curves and AUC values are computed for the three
    biomarkers per cohort
- name: qs
  role: Used to qread the cohort .qs dataset files into R before scoring and ROC analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:41157a7240f927f8daf353f4c2b48ab01e172c636398db7c165a91f8949c6f7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - iMLGAM
  - qs
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# risk-score-validation-across-cohorts

## Summary

Benchmark a computed risk score (e.g., the iMLGAM riskscore) against established clinical biomarkers such as PD-L1 expression and TMB by comparing ROC curves and AUC values across multiple independent ICB cohorts (Mariathasan, Braun, OAK, Riaz).

## When to use

Use when a continuous predictive score has been computed for immunotherapy cohorts and you must demonstrate it outperforms existing biomarkers (PD-L1, TMB) via ROC/AUC comparison across cohorts.

## When NOT to use

- The score has not yet been computed — run iMLGAM Steps 1–4 (abess feature selection, basic_learner, GA_train, predict) first to obtain riskscore values
- The outcome variable is continuous survival time rather than the binary immunotherapy response 'res' — ROC/AUC comparison is for binary response classification
- Only a single cohort is available and no competing biomarkers (PD-L1, TMB) can be measured, precluding the comparative validation this skill performs

## Inputs

- ICB cohort gene-pair expression datasets (.qs files, e.g. 'Sample File.qs', 'ICI cohort gene pairs.qs') with a binary 'res' response column
- Computed iMLGAM risk scores per sample (from the GA model via predict(ga_fit, ...))
- Per-sample PD-L1 expression values
- Per-sample TMB values

## Outputs

- roc_auc_comparison.csv table of per-cohort AUC values for iMLGAM score, PD-L1, and TMB
- Per-cohort ROC curve figures comparing all three biomarkers

## How to apply

Load each ICB cohort dataset in R with the qs package (qread on 'Sample File.qs' and 'ICI cohort gene pairs.qs'), setting the binary immunotherapy response column 'res' as the outcome as in the iMLGAM Step 4 scoring script. For each cohort, compute ROC curves and AUC values for three continuous predictors — the iMLGAM riskscore (from predict(ga_fit, ...)), PD-L1 expression, and TMB — using the binary 'res' response as outcome. Combine per-cohort AUCs into a single comparison table (roc_auc_comparison.csv) and verify the iMLGAM AUC exceeds both PD-L1 and TMB in every cohort. Plot the ROC curves for all three biomarkers per cohort and save the figures alongside the comparison table.

## Related tools

- **iMLGAM** (R package providing GA_train and predict to compute the riskscore that is benchmarked against PD-L1 and TMB) — https://github.com/Yelab1994/iMLGAM
- **R** (Environment in which ROC curves and AUC values are computed for the three biomarkers per cohort)
- **qs** (Used to qread the cohort .qs dataset files into R before scoring and ROC analysis)

## Examples

```
library(qs); dat <- qread("ICI cohort gene pairs.qs"); validatres$riskscore <- predict(ga_fit, validatres)
```

## Evaluation signals

- In every cohort (Mariathasan, Braun, OAK, Riaz), the iMLGAM AUC is strictly greater than both the PD-L1 AUC and the TMB AUC
- Each cohort's ROC analysis uses the same binary outcome ('res') for all three predictors, so AUCs are directly comparable within a cohort
- roc_auc_comparison.csv contains one row per cohort per predictor (or per cohort) with numeric AUC values, no missing entries
- Risk scores are finite and non-degenerate (non-constant across samples) before ROC computation
- Saved ROC figures show three curves (iMLGAM, PD-L1, TMB) per cohort with AUC values annotated

## Limitations

- The 11 RNA-seq cohorts analyzed cover only six cancer types; pan-cancer predictive efficacy of the iMLGAM score requires further validation through prospective ICB trials
- No large-scale, multi-center pan-cancer cohort validation currently exists, limiting the model's generalizability
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses

## Evidence

- [results] The iMLGAM score consistently exceeded both PD-L1 and TMB in ROC curve analyses across the Mariathasan, Braun, OAK, and Riaz cohorts.: [evidence span withheld — anchor status: not_found]
- [readme] dat <- qread("ICI cohort gene pairs.qs"): "dat <- qread("ICI cohort gene pairs.qs")"
- [abstract] iMLGAM scores exhibit superior predictive performance across independent cohorts, with lower scores correlating significantly with enhanced therapeutic responses and outperforming existing clinical: "iMLGAM scores exhibit superior predictive performance across independent cohorts, with lower scores correlating significantly with enhanced therapeutic responses and outperforming existing clinical"
- [readme] testres$riskscore<- predict(ga_fit ,testres): "testres$riskscore<- predict(ga_fit ,testres)"
- [discussion] the 11 RNA-seq cohorts analyzed represent a limited range of six cancer types, with the iMLGAM score's pan-cancer predictive efficacy requiring further validation through prospective ICB trials: "the 11 RNA‐seq cohorts analyzed represent a limited range of six cancer types, with the iMLGAM score's pan‐cancer predictive efficacy requiring further validation through prospective ICB trials"
