---
name: immunotherapy-cohort-response-annotation
description: Use when cohort-level RNA-seq/ICI datasets with a binary immunotherapy
  response column ('res'), PD-L1 expression, and TMB are available and the goal is
  to test whether the iMLGAM score outperforms existing biomarkers.
when_to_use_negative:
- Input is raw sequencing data or a feature matrix lacking the binary 'res' immunotherapy
  response outcome — score computation and ROC benchmarking cannot proceed without
  annotated cohorts
- The task is training or refitting the iMLGAM model itself (base learner construction,
  ABESS selection, genetic algorithm) rather than evaluating an already-computed score
  against existing biomarkers
- Only one predictor is available per cohort — the skill's purpose is a three-way
  biomarker comparison (iMLGAM vs PD-L1 vs TMB), not single-biomarker evaluation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: iMLGAM
  role: R package providing the 4-step scoring pipeline (basic_learner, basic_learner_predictor,
    GA_train, predict) that produces the riskscore benchmarked against PD-L1 and TMB
  repo: https://github.com/Yelab1994/iMLGAM
- name: qs
  role: Used to load the cohort .qs files ('Sample File.qs', 'ICI cohort gene pairs.qs')
    containing the ICI cohort gene pairs and 'res' outcome columns
- name: abess
  role: Feature selection (abess_fit <- abess(trainset[,-1], trainset$res, support.size=0:100))
    that determines the key genes feeding the base learners whose predictions form
    the riskscore
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:23d8f708e660044f96b464d81531f18778491a6d5662a007de7b851b870eddea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - abess
  - iMLGAM
  - qs
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# immunotherapy-cohort-response-annotation

## Summary

Benchmark the iMLGAM riskscore against PD-L1 expression and TMB as predictors of binary ICB response ('res') in independent immunotherapy cohorts (Mariathasan, Braun, OAK, Riaz) using ROC/AUC analysis in R, producing a per-cohort AUC comparison table and ROC figures.

## When to use

Use when cohort-level RNA-seq/ICI datasets with a binary immunotherapy response column ('res'), PD-L1 expression, and TMB are available and the goal is to test whether the iMLGAM score outperforms existing biomarkers.

## When NOT to use

- Input is raw sequencing data or a feature matrix lacking the binary 'res' immunotherapy response outcome — score computation and ROC benchmarking cannot proceed without annotated cohorts
- The task is training or refitting the iMLGAM model itself (base learner construction, ABESS selection, genetic algorithm) rather than evaluating an already-computed score against existing biomarkers
- Only one predictor is available per cohort — the skill's purpose is a three-way biomarker comparison (iMLGAM vs PD-L1 vs TMB), not single-biomarker evaluation

## Inputs

- ICI cohort datasets (e.g., Mariathasan, Braun, OAK, Riaz) as .qs files loaded with qread, with a binary response column 'res' and sample row names
- Computed iMLGAM riskscore per sample (output of predict(ga_fit, ...) after the 4-step iMLGAM pipeline)
- PD-L1 expression values per sample
- Tumor mutational burden (TMB) values per sample

## Outputs

- roc_auc_comparison.csv table of per-cohort AUC values for iMLGAM score, PD-L1, and TMB
- Per-cohort ROC curve figures comparing the three biomarkers

## How to apply

Load each cohort's data in R via the qs package with qread on the 'Sample File.qs' and 'ICI cohort gene pairs.qs' files, keeping the first column as the binary outcome ('res'). Compute iMLGAM riskscores per cohort using the Step 4 scoring formula: trainres$riskscore <- predict(ga_fit, trainres), where ga_fit is the genetic-algorithm-trained ensemble (GA_train on base-learner prediction matrices built after ABESS feature selection with support.size=0:100). For each cohort, run ROC curve and AUC analyses in R for three continuous predictors — iMLGAM riskscore, PD-L1 expression, and TMB — against the binary 'res' outcome. Combine per-cohort AUC values into a single roc_auc_comparison.csv and verify that the iMLGAM AUC exceeds both PD-L1 and TMB AUCs in every cohort. Plot ROC curves for all three biomarkers per cohort and save the figures alongside the CSV.

## Related tools

- **iMLGAM** (R package providing the 4-step scoring pipeline (basic_learner, basic_learner_predictor, GA_train, predict) that produces the riskscore benchmarked against PD-L1 and TMB) — https://github.com/Yelab1994/iMLGAM
- **qs** (Used to load the cohort .qs files ('Sample File.qs', 'ICI cohort gene pairs.qs') containing the ICI cohort gene pairs and 'res' outcome columns)
- **abess** (Feature selection (abess_fit <- abess(trainset[,-1], trainset$res, support.size=0:100)) that determines the key genes feeding the base learners whose predictions form the riskscore)

## Examples

```
library(qs); dat <- qread("ICI cohort gene pairs.qs"); testres$riskscore <- predict(ga_fit, testres)
```

## Evaluation signals

- In every cohort (Mariathasan, Braun, OAK, Riaz), the iMLGAM score AUC exceeds both the PD-L1 AUC and the TMB AUC in the ROC comparison
- roc_auc_comparison.csv is produced with AUC values for all three predictors in all four cohorts, each AUC within [0,1]
- A riskscore column is populated for each cohort via predict(ga_fit, ...) on the GA-trained ensemble, and lower scores correspond to responder-enriched samples
- Per-cohort ROC figures are saved showing the three biomarkers' curves against the binary 'res' outcome
- The 'res' column is set identically to the iMLGAM Step 4 scoring script after stripping columns 1, 3, and 4 of the loaded .qs data (colnames(...)[1] <- "res")

## Limitations

- The 11 RNA-seq cohorts analyzed represent only six cancer types; pan-cancer predictive efficacy requires further validation in prospective ICB trials
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses
- No large-scale, multi-center pan-cancer cohort validation exists, limiting the model's generalizability

## Evidence

- [abstract] iMLGAM scores exhibit superior predictive performance across independent cohorts, with lower scores correlating significantly with enhanced therapeutic responses and outperforming existing clinical: "iMLGAM scores exhibit superior predictive performance across independent cohorts, with lower scores correlating significantly with enhanced therapeutic responses and outperforming existing clinical"
- [methods] dat <- qread("ICI cohort gene pairs.qs"): "dat <- qread("ICI cohort gene pairs.qs")"
- [methods] abess_fit <- abess(trainset[,-1], trainset$res,support.size=0:100): "abess_fit <- abess(trainset[,-1], trainset$res,support.size=0:100)"
- [readme] The first column is the outcome (with row names as 'res'), and the subsequent columns are features.: "The first column is the outcome (with row names as 'res'), and the subsequent columns are features."
- [readme] testres$riskscore<- predict(ga_fit ,testres): "testres$riskscore<- predict(ga_fit ,testres)"
