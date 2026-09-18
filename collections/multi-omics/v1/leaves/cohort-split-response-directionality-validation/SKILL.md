---
name: cohort-split-response-directionality-validation
description: Use when a fitted GA_train model must be applied to held-out ICB cohort
  splits to score patients and check that the risk score discriminates ICB responders
  with the expected directionality.
when_to_use_negative:
- 'The GA model (ga_fit) has not yet been fitted — upstream base-learner construction
  (Steps 1–3: abess feature selection, basic_learner, GA_train) must be completed
  first'
- The goal is unsupervised patient stratification without response labels ('res');
  the directionality check requires responder/non-responder annotations
- The input is raw transcriptomic data rather than the gene-pair prediction matrices
  produced by basic_learner_predictor
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0203
tools:
- name: iMLGAM
  role: Provides GA_train and predict for fitting the genetic-algorithm ensemble and
    computing riskscores per cohort split
  repo: https://github.com/Yelab1994/iMLGAM
- name: qs
  role: Reads the 'Sample File.qs' and 'ICI cohort gene pairs.qs' files that supply
    response labels and prediction matrices
- name: R
  role: Runtime environment for the scoring and AUC evaluation workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:fdc323e78daee55928305ff7d198512dd4a5b646bbf9d340fc20c1a5044d6590
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
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# Cohort-split response directionality validation

## Summary

Scores the 70% training, 30% validation, and test ICB cohorts with the GA-fitted iMLGAM ensemble model and verifies that lower riskscores discriminate responders (ROC-AUC ≈ 0.76 train, 0.72 validation, 0.70 test, all p < 0.001).

## When to use

Use when a fitted GA_train model must be applied to held-out ICB cohort splits to score patients and check that the risk score discriminates ICB responders with the expected directionality.

## When NOT to use

- The GA model (ga_fit) has not yet been fitted — upstream base-learner construction (Steps 1–3: abess feature selection, basic_learner, GA_train) must be completed first
- The goal is unsupervised patient stratification without response labels ('res'); the directionality check requires responder/non-responder annotations
- The input is raw transcriptomic data rather than the gene-pair prediction matrices produced by basic_learner_predictor

## Inputs

- Prediction matrices trainres, validatres, and testres with base-learner model predictions from basic_learner_predictor (R data frames)
- Response-labelled data frames from 'Sample File.qs' and 'ICI cohort gene pairs.qs' containing the 'res' outcome column (loaded with the qs package)
- GA-fitted ensemble model object ga_fit from GA_train()

## Outputs

- Scored trainres, validatres, and testres data frames with an appended 'riskscore' column
- Per-cohort ROC-AUC values for the training, testing, and validation sets (~0.76 / ~0.70 / ~0.72)
- A combined risk-score table (e.g. imlgam_risk_scores.csv) for all three cohorts

## How to apply

Load the response-labelled prediction matrices (trainres, validatres, testres) built in Steps 2–3 of the iMLGAM README from 'Sample File.qs' and 'ICI cohort gene pairs.qs' with the qs package, keeping only the 'res' outcome column and intersecting row names across the two data sources. Fit or load the genetic-algorithm-optimized model via ga_fit <- GA_train(trainres), then append a 'riskscore' column to each split by calling predict(ga_fit, ...) on trainres, validatres, and testres. Compute ROC-AUC for each split against the 'res' labels and confirm that responders have significantly lower riskscores, since the iMLGAM score is constructed so lower scores indicate greater likelihood of ICB benefit. Accept the run when riskscore values are finite for all patients, per-split AUCs approach the reported targets (~0.76 train, ~0.70 test, ~0.72 validation), and the responder/non-responder score separation is significant (p < 0.001).

## Related tools

- **iMLGAM** (Provides GA_train and predict for fitting the genetic-algorithm ensemble and computing riskscores per cohort split) — https://github.com/Yelab1994/iMLGAM
- **qs** (Reads the 'Sample File.qs' and 'ICI cohort gene pairs.qs' files that supply response labels and prediction matrices)
- **R** (Runtime environment for the scoring and AUC evaluation workflow)

## Examples

```
library(qs); library(iMLGAM); ga_fit <- GA_train(trainres); trainres$riskscore <- predict(ga_fit, trainres)
```

## Evaluation signals

- riskscore values are finite for all patients in each of the three splits (train, validation, test)
- Per-split ROC-AUC matches reported discrimination levels: ~0.76 (train), ~0.70 (test), ~0.72 (validation)
- Responders show significantly lower riskscores than non-responders (all p < 0.001), confirming score directionality
- Row-name intersection across 'Sample File.qs' and 'ICI cohort gene pairs.qs' prediction matrices yields matched samples in all three splits

## Limitations

- The 11 RNA-seq cohorts cover only six cancer types; pan-cancer predictive efficacy requires validation in prospective ICB trials
- No large-scale, multi-center pan-cancer cohort validation exists, limiting generalizability of the score
- Some datasets lack critical clinical annotations, which can hinder downstream biomarker analyses

## Evidence

- [methods] AUC targets: "trainres $riskscore <- predict(ga_fit ,trainres)"
- [abstract] cohort directionality: [evidence span withheld — anchor status: not_found]
- [intro] score computation scope: "an R package and its novel scoring system for predicting outcomes in cancer patients undergoing ICB"
- [results] per-split AUC: [evidence span withheld — anchor status: not_found]
- [readme] package installation: "devtools::install_github("Yelab1994/iMLGAM")"
