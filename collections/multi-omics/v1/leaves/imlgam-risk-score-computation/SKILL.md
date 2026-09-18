---
name: imlgam-risk-score-computation
description: Use when a GA-trained iMLGAM model (ga_fit) exists and per-patient ICB
  response risk scores must be computed from training (70%), validation (30%), and
  test prediction matrices, then evaluated for discrimination (ROC-AUC) against the
  'res' outcome labels.
when_to_use_negative:
- No GA-fitted model (ga_fit) is available — scoring requires the upstream GA_train
  ensemble fit; raw gene-pair expression alone cannot be scored directly
- The goal is feature selection or base-learner construction (abess support.size=0:100,
  basic_learner) rather than applying an already-trained model to new cohorts
- Outcome data lack the ICB response labels ('res' column), so riskscores cannot be
  evaluated for discrimination or directionality
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3577
tools:
- name: iMLGAM
  role: Provides GA_train and predict() used to fit the genetic-algorithm-optimized
    ensemble and compute riskscores for each cohort
  repo: https://github.com/Yelab1994/iMLGAM
- name: iMLAGA
  role: Underlying R package dependency installed from GitHub that powers the iMLGAM
    genetic-algorithm workflow
  repo: https://github.com/Yelab1994/iMLAGA
- name: qs
  role: Loads the 'Sample File.qs' and 'ICI cohort gene pairs.qs' prediction/outcome
    matrices via qread()
- name: R
  role: Runtime environment in which the scoring, AUC computation, and responder/non-responder
    comparisons are performed
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:b78c94a98a1c029f07bf0467db42b97fdb50a3c72cc1e5793d86bcd95939e0f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - devtools
  - iMLAGA
  - iMLGAM
  - qs
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2222222222222222
n_claims: 9
---

# imlgam-risk-score-computation

## Summary

Computes per-patient iMLGAM immunotherapy risk scores by applying the genetic-algorithm-optimized ensemble model (GA_train/ga_fit) to train, validation, and test base-learner prediction matrices, yielding a single riskscore per patient that discriminates ICB responders (lower scores) from non-responders.

## When to use

Use when a GA-trained iMLGAM model (ga_fit) exists and per-patient ICB response risk scores must be computed from training (70%), validation (30%), and test prediction matrices, then evaluated for discrimination (ROC-AUC) against the 'res' outcome labels.

## When NOT to use

- No GA-fitted model (ga_fit) is available — scoring requires the upstream GA_train ensemble fit; raw gene-pair expression alone cannot be scored directly
- The goal is feature selection or base-learner construction (abess support.size=0:100, basic_learner) rather than applying an already-trained model to new cohorts
- Outcome data lack the ICB response labels ('res' column), so riskscores cannot be evaluated for discrimination or directionality

## Inputs

- 'Sample File.qs' with 70% training (trainset_7), 30% validation (trainset_3), and testset splits
- 'ICI cohort gene pairs.qs' with ICI cohort outcome annotations ('res' column)
- Base-learner prediction matrices (trainres, validatres, testres) from basic_learner_predictor
- GA-fitted ensemble model (ga_fit) from GA_train on the training prediction matrix

## Outputs

- Per-patient riskscore column appended to trainres, validatres, and testres
- Per-split ROC-AUC values (targets ~0.76 train, ~0.70 test, ~0.72 validation)
- Combined scored table imlgam_risk_scores.csv

## How to apply

Load the response-labelled prediction matrices from 'Sample File.qs' and 'ICI cohort gene pairs.qs' with qread(), keeping the outcome column 'res' and intersecting row names between the outcome file and the basic_learner_predictor outputs (trainres, validatres, testres). Fit the genetic-algorithm ensemble once on the training prediction matrix via ga_fit <- GA_train(trainres); this step selects the optimal combination of base learners built with basic_learner (10-fold CV, 5 repeats, 8 cores). Score each split by appending a 'riskscore' column via predict(ga_fit, ...) on trainres, validatres, and testres — no refitting is done on validation or test data. Compute ROC-AUC of riskscore versus 'res' per split and check directionality: lower riskscores should be enriched in responders. Assemble the three scored cohorts into imlgam_risk_scores.csv; accept the run if riskscores are finite for all patients and per-split AUCs reach the reported levels (~0.76 train, ~0.70 test, ~0.72 validation) with responders showing significantly lower scores (p < 0.001).

## Related tools

- **iMLGAM** (Provides GA_train and predict() used to fit the genetic-algorithm-optimized ensemble and compute riskscores for each cohort) — https://github.com/Yelab1994/iMLGAM
- **iMLAGA** (Underlying R package dependency installed from GitHub that powers the iMLGAM genetic-algorithm workflow) — https://github.com/Yelab1994/iMLAGA
- **qs** (Loads the 'Sample File.qs' and 'ICI cohort gene pairs.qs' prediction/outcome matrices via qread())
- **R** (Runtime environment in which the scoring, AUC computation, and responder/non-responder comparisons are performed)

## Examples

```
trainres$riskscore <- predict(ga_fit, trainres); validatres$riskscore <- predict(ga_fit, validatres); testres$riskscore <- predict(ga_fit, testres)
```

## Evaluation signals

- riskscore values are finite for all patients in each split (no NA/NaN after predict(ga_fit, ...))
- Per-split ROC-AUC matches reported levels: ~0.76 training, ~0.70 testing, ~0.72 validation (all p < 0.001)
- Responders show significantly lower riskscores than non-responders in every split, confirming score directionality
- Tumor-type stratification yields AUC > 0.6 with lower scores in responders for RCC, urothelial carcinoma, melanoma, and gastric cancer; GBM shows no significant responder/non-responder difference
- Row names of scored matrices intersect correctly with 'ICI cohort gene pairs.qs' outcome annotations (cbind of 'res' with prediction matrices succeeds for all three splits)

## Limitations

- GBM showed no significant responder/non-responder difference, likely due to limited sample size
- The 11 RNA-seq cohorts cover only six cancer types; pan-cancer predictive efficacy requires validation in prospective ICB trials and large-scale multi-center cohorts
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses

## Evidence

- [methods] GA fit on training prediction matrix: "ga_fit <- GA_train(trainres)"
- [readme] Score assignment via predict on the test set: "testres$riskscore<- predict(ga_fit ,testres)"
- [abstract] Reported discrimination performance of the iMLGAM score: "iMLGAM scores exhibit superior predictive performance across independent cohorts, with lower scores correlating significantly with enhanced therapeutic responses"
- [discussion] Directionality of the score and immune phenotype of low scorers: "specimens with reduced iMLGAM scores exhibited an inflammatory immunological profile, marked by enhanced CD8+T lymphocyte infiltration and elevated immunogenicity"
