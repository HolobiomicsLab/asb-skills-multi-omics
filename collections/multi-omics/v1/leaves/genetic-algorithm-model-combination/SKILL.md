---
name: genetic-algorithm-model-combination
description: Use when base-learner prediction matrices for training/validation/test
  sets already exist (from basic_learner_predictor) and an optimized ensemble scoring
  formula must be derived and applied to compute riskscores.
when_to_use_negative:
- Input is raw expression data rather than base-learner prediction matrices — run
  abess feature selection and basic_learner first
- Only a single machine-learning model is available, since the genetic algorithm optimizes
  a combination of multiple base learners
- No response outcome ('res') annotation is available for the ICI cohort to merge
  with the prediction matrices
tools:
- name: iMLGAM
  role: R package providing GA_train and predict to optimize the base-learner combination
    and score samples
  repo: https://github.com/Yelab1994/iMLGAM
- name: iMLAGA
  role: Required dependency implementing the genetic learning algorithm installed
    from GitHub
  repo: https://github.com/Yelab1994/iMLAGA
- name: qs
  role: Loads 'ICI cohort gene pairs.qs' whose 'res' outcome is merged onto the prediction
    matrices
- name: devtools
  role: Used to install the iMLGAM package via install_github
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:443f6544ae09527c72344096a2f7956b3844185a4ab72525f64d1a93f24b4f51
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 9
---

# genetic-algorithm-model-combination

## Summary

Applies the iMLGAM/iMLAGA genetic-algorithm optimization (GA_train) to an ensemble of base-learner prediction matrices, selecting the optimal combination of machine-learning models and converting it into a riskscore for ICB response prediction.

## When to use

Use when base-learner prediction matrices for training/validation/test sets already exist (from basic_learner_predictor) and an optimized ensemble scoring formula must be derived and applied to compute riskscores.

## When NOT to use

- Input is raw expression data rather than base-learner prediction matrices — run abess feature selection and basic_learner first
- Only a single machine-learning model is available, since the genetic algorithm optimizes a combination of multiple base learners
- No response outcome ('res') annotation is available for the ICI cohort to merge with the prediction matrices

## Inputs

- trainres: training-set prediction matrix from basic_learner_predictor
- validatres: validation-set prediction matrix from basic_learner_predictor
- testres: test-set prediction matrix from basic_learner_predictor
- ICI cohort gene pairs.qs (loaded via qs::qread) providing the 'res' outcome column to merge by rownames

## Outputs

- ga_fit: fitted genetic-algorithm model object with the optimal base-learner combination
- trainres$riskscore, validatres$riskscore, testres$riskscore: per-sample risk scores for each set

## How to apply

First construct the prediction matrices with basic_learner_predictor for the training (trainset_7), validation (trainset_3), and test sets, then merge each with the ICI cohort outcomes ('res' column) from 'ICI cohort gene pairs.qs' loaded via qread, matching samples by intersect(rownames(...)). Run GA_train on the training prediction matrix only (ga_fit <- GA_train(trainres)) so the genetic algorithm selects the optimal combination of base learners without test-set leakage. Apply predict(ga_fit, ...) to the training, validation, and test matrices to obtain the riskscore column for each set. The rationale is that ensemble learning improves predictive accuracy over individual learners, while the GA automates base-learner combination and guards against overfitting on heterogeneous sequencing platforms.

## Related tools

- **iMLGAM** (R package providing GA_train and predict to optimize the base-learner combination and score samples) — https://github.com/Yelab1994/iMLGAM
- **iMLAGA** (Required dependency implementing the genetic learning algorithm installed from GitHub) — https://github.com/Yelab1994/iMLAGA
- **qs** (Loads 'ICI cohort gene pairs.qs' whose 'res' outcome is merged onto the prediction matrices)
- **devtools** (Used to install the iMLGAM package via install_github)

## Examples

```
ga_fit <- GA_train(trainres); trainres$riskscore <- predict(ga_fit, trainres); testres$riskscore <- predict(ga_fit, testres)
```

## Evaluation signals

- ga_fit is trained only on trainres (training prediction matrix), with validation/test scores generated via predict() — no leakage into GA optimization
- Each prediction matrix gains a numeric 'riskscore' column after predict(ga_fit, ...) for all three sets (train, validation, test)
- Downstream validation: riskscore should be lower in ICB responders (p < 0.05, Wilcoxon rank-sum) with ROC-AUC > 0.6 in RCC, urothelial carcinoma, melanoma, and gastric cancer
- Row counts of merged matrices equal the intersect of rownames between the .qs cohort file and each prediction matrix

## Limitations

- The 11 RNA-seq cohorts cover only six cancer types; no significant responder/non-responder difference was seen in GBM, likely due to limited sample size
- Lack of large-scale, multi-center pan-cancer cohort validation limits generalizability; the GA overfitting risk from heterogeneous sequencing platforms is what the automated combination is meant to mitigate
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses

## Evidence

- [methods] Step 3: Running a genetic learning algorithm to obtain the optimal combination of base learners: "## Step 3: Running a genetic learning algorithm to obtain the optimal combination of base learners"
- [readme] GA_train call: "ga_fit <- GA_train(trainres)"
- [readme] risk score prediction on training set: "trainres $riskscore <- predict(ga_fit ,trainres)"
- [intro] rationale for ensemble learning: "ensemble learning methodologies have been demonstrated to provide enhanced predictive accuracy for outcomes when compared to individual machine learning approaches"
