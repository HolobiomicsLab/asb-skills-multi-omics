---
name: auc-comparison-across-datasets
description: Use when you have a continuous risk score plus one or more competing
  biomarkers measured on the same samples in multiple immunotherapy cohorts with a
  binary response column ('res'), and need a per-cohort AUC comparison table.
when_to_use_negative:
- Input data lacks a binary response column ('res') or the competing biomarkers (PD-L1,
  TMB) are not measured on the same samples, making paired AUC comparison impossible
- The goal is model training or feature selection (use the iMLGAM Step 1–4 pipeline
  with abess, basic_learner, and GA_train instead of benchmarking an already-computed
  score)
- Cohorts are not ICB-treated immunotherapy cohorts — predictive performance of the
  iMLGAM score is specific to ICB response and its pan-cancer efficacy still requires
  prospective validation
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: iMLGAM
  role: R package supplying the scoring system (GA-trained model via predict/ga_fit)
    whose riskscore is benchmarked against PD-L1 and TMB
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Environment for computing ROC curves/AUC values per cohort and producing the
    comparison table and figures
- name: qs
  role: R package used to load cohort datasets via qread from .qs files (Sample File.qs,
    ICI cohort gene pairs.qs)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:052c48803ce6c64ea45258005132af67228f24fc013e10caa736f5ad6fcbc0bb
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# auc-comparison-across-datasets

## Summary

Benchmark a continuous predictive score (e.g., iMLGAM riskscore) against established biomarkers (PD-L1 expression, TMB) by computing ROC curves and AUC values per cohort and verifying the new score's AUC exceeds the comparators in every dataset.

## When to use

Use when you have a continuous risk score plus one or more competing biomarkers measured on the same samples in multiple immunotherapy cohorts with a binary response column ('res'), and need a per-cohort AUC comparison table.

## When NOT to use

- Input data lacks a binary response column ('res') or the competing biomarkers (PD-L1, TMB) are not measured on the same samples, making paired AUC comparison impossible
- The goal is model training or feature selection (use the iMLGAM Step 1–4 pipeline with abess, basic_learner, and GA_train instead of benchmarking an already-computed score)
- Cohorts are not ICB-treated immunotherapy cohorts — predictive performance of the iMLGAM score is specific to ICB response and its pan-cancer efficacy still requires prospective validation

## Inputs

- Cohort expression/response datasets loaded from .qs files (Sample File.qs, ICI cohort gene pairs.qs) for Mariathasan, Braun, OAK, and Riaz ICB cohorts
- Computed iMLGAM riskscore per sample (from Step 4: predict(ga_fit, ...))
- PD-L1 expression values per sample
- Tumor mutational burden (TMB) values per sample
- Binary immunotherapy response column 'res'

## Outputs

- roc_auc_comparison.csv — table of per-cohort AUC values for iMLGAM score, PD-L1, and TMB
- Per-cohort ROC curve figures plotting all three biomarkers
- Boolean check that iMLGAM AUC > PD-L1 AUC and iMLGAM AUC > TMB AUC in every cohort

## How to apply

Load each ICB cohort dataset in R with the qs package (qread on 'Sample File.qs' and 'ICI cohort gene pairs.qs'), keeping the binary 'res' column as the outcome exactly as set in the iMLGAM Step 4 scoring script. For each cohort (e.g., Mariathasan, Braun, OAK, Riaz), compute ROC curves and AUC values in R for three continuous predictors side by side: the iMLGAM riskscore obtained via predict(ga_fit, ...), PD-L1 expression, and TMB, using 'res' as the outcome. Combine the per-cohort AUC values into a single comparison table (roc_auc_comparison.csv) and check the invariant that the iMLGAM score AUC exceeds both the PD-L1 and TMB AUCs in every cohort. Finally plot the ROC curves for all three biomarkers per cohort and save the figures alongside the comparison table.

## Related tools

- **iMLGAM** (R package supplying the scoring system (GA-trained model via predict/ga_fit) whose riskscore is benchmarked against PD-L1 and TMB) — https://github.com/Yelab1994/iMLGAM
- **R** (Environment for computing ROC curves/AUC values per cohort and producing the comparison table and figures)
- **qs** (R package used to load cohort datasets via qread from .qs files (Sample File.qs, ICI cohort gene pairs.qs))

## Examples

```
library(qs); dat <- qread("ICI cohort gene pairs.qs"); testres$riskscore <- predict(ga_fit, testres)
```

## Evaluation signals

- roc_auc_comparison.csv contains one row per cohort (Mariathasan, Braun, OAK, Riaz) with three AUC columns (iMLGAM, PD-L1, TMB) and no missing values
- Invariant holds: iMLGAM score AUC > PD-L1 AUC and iMLGAM AUC > TMB AUC in every cohort (the article's reported finding)
- Outcome variable in each ROC computation is the binary 'res' column, unmodified from the iMLGAM Step 4 scoring setup
- Per-cohort ROC figures plot all three biomarkers' curves on comparable axes and are saved alongside the CSV
- Riskscore values derive from predict(ga_fit, ...) on the same sample identifiers used for PD-L1/TMB comparison (sample-level pairing verified)

## Limitations

- The 11 RNA-seq cohorts analyzed represent a limited range of six cancer types, with the iMLGAM score's pan-cancer predictive efficacy requiring further validation through prospective ICB trials
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses
- No large-scale, multi-center pan-cancer cohort validation exists, limiting the model's generalizability beyond the analyzed cohorts

## Evidence

- [results] The iMLGAM score consistently exceeded both PD-L1 and TMB in ROC curve analyses across the Mariathasan, Braun, OAK, and Riaz cohorts.: [evidence span withheld — anchor status: not_found]
- [methods] qs qread: "dat <- qread("ICI cohort gene pairs.qs")"
- [readme] risk scores calculation: "#Obtaining the evaluation score of the test set
testres$riskscore<- predict(ga_fit ,testres)"
- [readme] outcome column definition: "#The first column is the outcome (with row names as 'res'), and the subsequent columns are features."
- [abstract] outperforming existing clinical biomarkers: "iMLGAM scores exhibit superior predictive performance across independent cohorts, with lower scores correlating significantly with enhanced therapeutic responses and outperforming existing clinical"
