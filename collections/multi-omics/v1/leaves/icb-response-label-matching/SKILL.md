---
name: icb-response-label-matching
description: 'Use when scoring ICB responder vs non-responder cohorts with a fitted
  iMLGAM ga_fit model: prediction matrices (trainres, validatres, testres) lack the
  outcome column and must be re-aligned to ''res'' labels by row-name intersection
  before predict() and ROC-AUC evaluation.'
when_to_use_negative:
- Input is already a label-aligned matrix that includes the 'res' column and matching
  row names — re-cbinding would duplicate the outcome column
- Working outside the iMLGAM workflow (e.g. predicting ICB response from a single-feature
  biomarker such as TMB or PD-L1 IHC) where no base-learner prediction matrix exists
  to align
- Datasets lacking response annotations ('res') for a cohort; the paper notes some
  datasets lack critical clinical annotations, making label matching impossible
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: iMLGAM
  role: R package providing basic_learner_predictor, GA_train, and predict used to
    build and score the prediction matrices that the 'res' labels are matched onto
  repo: https://github.com/Yelab1994/iMLGAM
- name: qs
  role: R package used to qread the 'Sample File.qs' and 'ICI cohort gene pairs.qs'
    files that supply the response labels and cohort row names
- name: R
  role: Language in which the label intersection (intersect of rownames, cbind) and
    downstream scoring are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:38c65a9e41db5c7f1c4fcc1225fa712bd90833457f7f78ff4e18b504c41c78d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - iMLGAM
  - qs
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# icb-response-label-matching

## Summary

A data-preparation step in the iMLGAM workflow that reattaches the ICB response outcome column ('res') to each cohort's base-learner prediction matrix by intersecting row names, so that risk scores from GA_train can be compared against ground-truth responder labels. Correct label matching is prerequisite for computing per-cohort ROC-AUC (train ~0.76, test ~0.70, validation ~0.72).

## When to use

Use when scoring ICB responder vs non-responder cohorts with a fitted iMLGAM ga_fit model: prediction matrices (trainres, validatres, testres) lack the outcome column and must be re-aligned to 'res' labels by row-name intersection before predict() and ROC-AUC evaluation.

## When NOT to use

- Input is already a label-aligned matrix that includes the 'res' column and matching row names — re-cbinding would duplicate the outcome column
- Working outside the iMLGAM workflow (e.g. predicting ICB response from a single-feature biomarker such as TMB or PD-L1 IHC) where no base-learner prediction matrix exists to align
- Datasets lacking response annotations ('res') for a cohort; the paper notes some datasets lack critical clinical annotations, making label matching impossible

## Inputs

- 'Sample File.qs' containing trainset_7, trainset_3, and testset lists with outcome column 'res' (read via qs::qread)
- 'ICI cohort gene pairs.qs' containing response labels for the ICI cohorts (read via qs::qread)
- Prediction matrices trainres, validatres, testres produced by basic_learner_predictor() from the iMLGAM package
- Fitted GA model ga_fit from GA_train(trainres) in the iMLGAM R package

## Outputs

- Label-aligned prediction matrices trainres, validatres, testres, each with a 'res' outcome column cbinded to the base-learner prediction columns
- Per-patient riskscore columns appended by predict(ga_fit, ...) for each split
- A combined scored table (e.g. imlgam_risk_scores.csv) with riskscore and res per patient per cohort

## How to apply

Load the cohort data ('Sample File.qs' and 'ICI cohort gene pairs.qs') with the qs package, dropping columns 1, 3, 4 and renaming the first column to 'res' so only the outcome column remains. For each split — dat$trainset_7 (70% training), dat$trainset_3 (30% validation), dat$testset — compute jj <- intersect(rownames(dat[[split]]), rownames(prediction_matrix)) and cbind the 'res' labels subset to those intersected rows onto the corresponding prediction matrix (trainres, validatres, testres). This step runs after basic_learner_predictor() has produced the per-model prediction matrices and immediately before GA_train(trainres), because the genetic algorithm and downstream predict(ga_fit, ...) need label-aligned matrices. Row-name intersection is the safeguard against sample-order drift between the qs files and the prediction matrices; never assume positional alignment. After scoring, verify directionality: lower riskscores should be enriched among responders (all p < 0.001 in the paper).

## Related tools

- **iMLGAM** (R package providing basic_learner_predictor, GA_train, and predict used to build and score the prediction matrices that the 'res' labels are matched onto) — https://github.com/Yelab1994/iMLGAM
- **qs** (R package used to qread the 'Sample File.qs' and 'ICI cohort gene pairs.qs' files that supply the response labels and cohort row names)
- **R** (Language in which the label intersection (intersect of rownames, cbind) and downstream scoring are executed)

## Examples

```
jj <- intersect(rownames(dat$testset),rownames(testres)); testres <- cbind(res=dat$testset[jj,]$res,testres[jj,]); testres$riskscore <- predict(ga_fit,testres)
```

## Evaluation signals

- riskscore values are finite (no NA/NaN) for all patients in each of the three splits after predict(ga_fit, ...)
- Per-split ROC-AUC reaches the reported discrimination levels: ~0.76 (train), ~0.70 (test), ~0.72 (validation), all p < 0.001
- Responders show significantly lower riskscores than non-responders in every cohort, confirming score directionality
- Number of rows in each label-aligned matrix equals the length of the row-name intersection jj for that split (no silently dropped or duplicated samples)
- Each aligned matrix has exactly one 'res' column followed by the base-learner prediction columns, mirroring the structure GA_train(trainres) was fitted on

## Limitations

- Only samples present in both the qs cohort file and the prediction matrix survive the row-name intersection; samples missing from either side are silently dropped
- The 11 RNA-seq cohorts cover only six cancer types, so the label-matched scores' pan-cancer predictive efficacy requires validation in prospective ICB trials
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses and label assignment

## Evidence

- [results] the iMLGAM score discriminated ICB responders from non-responders across all three cohorts (lower scores indicated greater likelihood of benefit, all p < 0.001), with ROC-AUC values of 0.76 for the training set, 0.70 for the testing set, and 0.72 for the validation set.: [evidence span withheld — anchor status: not_found]
- [readme] README Step 3 label alignment code: "jj <- intersect(rownames(dat$trainset_7),rownames(trainres))
trainres <- cbind(res=dat$trainset_7[jj,]$res,trainres[jj,])"
- [readme] README scoring step: "## Step 4: Calculating Risk Scores for Each set
Scoring each set using the formula constructed in Step 3."
- [methods] dataset loading from qs files: "dat <- qread("ICI cohort gene pairs.qs")"
