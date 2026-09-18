---
name: biomarker-predictive-performance-evaluation
description: Use when you have continuous biomarker scores for multiple immunotherapy
  cohorts with a binary response column ('res') and need to compare predictive performance
  (ROC/AUC) of the new biomarker against existing markers like PD-L1 or TMB.
when_to_use_negative:
- Outcome is continuous (e.g. survival time) rather than the binary immunotherapy
  response column ('res') — ROC/AUC comparison is inappropriate; use survival analysis
  instead
- No comparator biomarkers (PD-L1 expression or TMB) are available for the cohort,
  so no benchmarking table can be constructed
- The iMLGAM riskscores have not yet been computed (Steps 1–4 of the iMLGAM pipeline
  must run first to produce riskscore values via predict(ga_fit, ...))
edam_operation: http://edamontology.org/operation_1224
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0203
tools:
- name: iMLGAM
  role: Source of the riskscore predictor being benchmarked against PD-L1 and TMB
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Language in which ROC curves and AUC values are computed for the three predictors
    per cohort
- name: qs
  role: Loads the cohort .qs files (qread) containing the 'res' outcome column and
    gene-pair features
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:f34685bdfb9712fe9792423192c1fac845d886c8a2fe04ecc5d93c8119530490
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - iMLGAM
  - qs
  edam_operation: http://edamontology.org/operation_1224
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# biomarker-predictive-performance-evaluation

## Summary

Benchmark a continuous predictive biomarker against established clinical markers (e.g. PD-L1 expression, TMB) by computing ROC curves and AUC values per cohort in R, as done when comparing the iMLGAM risk score against PD-L1 and TMB in the Mariathasan, Braun, OAK, and Riaz ICB cohorts.

## When to use

Use when you have continuous biomarker scores for multiple immunotherapy cohorts with a binary response column ('res') and need to compare predictive performance (ROC/AUC) of the new biomarker against existing markers like PD-L1 or TMB.

## When NOT to use

- Outcome is continuous (e.g. survival time) rather than the binary immunotherapy response column ('res') — ROC/AUC comparison is inappropriate; use survival analysis instead
- No comparator biomarkers (PD-L1 expression or TMB) are available for the cohort, so no benchmarking table can be constructed
- The iMLGAM riskscores have not yet been computed (Steps 1–4 of the iMLGAM pipeline must run first to produce riskscore values via predict(ga_fit, ...))

## Inputs

- R data objects (.qs files, e.g. 'Sample File.qs', 'ICI cohort gene pairs.qs') containing per-cohort expression data with binary 'res' outcome column
- Per-sample iMLGAM riskscore values computed via predict(ga_fit, set)
- Continuous PD-L1 expression values per sample
- Continuous TMB values per sample
- Cohort labels for the Mariathasan, Braun, OAK, and Riaz ICB cohorts

## Outputs

- roc_auc_comparison.csv with per-cohort AUC values for iMLGAM score, PD-L1, and TMB
- Per-cohort ROC curve figures for all three biomarkers

## How to apply

Load each cohort dataset (e.g. via qs::qread on 'Sample File.qs' and 'ICI cohort gene pairs.qs') and attach the iMLGAM riskscore computed with predict(ga_fit, ...) as in the Step 4 scoring script. For each cohort, treat the binary immunotherapy response column ('res') as the outcome and compute ROC curves and AUC values in R for three continuous predictors: iMLGAM riskscore, PD-L1 expression, and TMB. Combine the per-cohort AUC values into a single comparison table (roc_auc_comparison.csv) and verify the iMLGAM AUC exceeds both PD-L1 and TMB AUCs in every cohort. Plot the per-cohort ROC curves for all three biomarkers and save the figures alongside the comparison table.

## Related tools

- **iMLGAM** (Source of the riskscore predictor being benchmarked against PD-L1 and TMB) — https://github.com/Yelab1994/iMLGAM
- **R** (Language in which ROC curves and AUC values are computed for the three predictors per cohort)
- **qs** (Loads the cohort .qs files (qread) containing the 'res' outcome column and gene-pair features)

## Examples

```
library(qs); dat <- qread("ICI cohort gene pairs.qs"); testres$riskscore <- predict(ga_fit, testres)
```

## Evaluation signals

- roc_auc_comparison.csv contains one AUC row per cohort (Mariathasan, Braun, OAK, Riaz) for each of the three predictors
- In every cohort, the iMLGAM AUC exceeds both the PD-L1 AUC and the TMB AUC, as reported in the paper
- ROC curves for all three biomarkers are plotted and saved per cohort alongside the table
- Each cohort's binary 'res' column is used consistently as the classification outcome across all three predictors

## Limitations

- The 11 RNA-seq cohorts analyzed cover only six cancer types; pan-cancer predictive efficacy of the iMLGAM score requires further validation through prospective ICB trials
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses
- There is no large-scale, multi-center pan-cancer cohort validation, limiting the model's generalizability

## Evidence

- [abstract] ROC/AUC comparison across four cohorts: "iMLGAM scores exhibit superior predictive performance across independent cohorts, with lower scores correlating significantly with enhanced therapeutic responses and outperforming existing clinical"
- [readme] Risk score computation endpoint: "testres$riskscore<- predict(ga_fit ,testres)"
- [readme] Binary outcome column and qread input format: "dat <- qread("Sample File.qs")
#The first column is the outcome (with row names as 'res'), and the subsequent columns are features."
- [methods] R environment requirement: "Make sure you have R and the following packages installed: iMLAGA."
- [full_text] Comparison scope cohorts: [evidence span withheld — anchor status: not_found]
