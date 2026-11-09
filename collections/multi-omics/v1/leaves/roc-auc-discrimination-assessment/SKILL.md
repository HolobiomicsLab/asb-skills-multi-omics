---
name: roc-auc-discrimination-assessment
description: Use when a fitted ga_fit (GA_train output) exists and you must score
  train/validat/test prediction matrices and quantify ICB-response discrimination
  with ROC-AUC per cohort.
when_to_use_negative:
- You are still constructing base learners or selecting features with ABESS — scoring
  requires a ga_fit model from GA_train (Step 3 output), not the raw gene-pair training
  set
- The outcome labels ('res') are missing or misaligned — row names of prediction matrices
  must be intersected with the gene-pair cohort data before scoring, otherwise AUC
  is meaningless
- The task is prognostic (survival/time-to-event) evaluation rather than binary ICB
  responder/non-responder discrimination
tools:
- name: iMLGAM
  role: R package providing GA_train and predict() to fit the genetic-algorithm model
    and compute riskscores per cohort
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: runtime environment for the scoring and ROC-AUC evaluation workflow
- name: qs
  role: loads the 'Sample File.qs' and 'ICI cohort gene pairs.qs' cohort/prediction-matrix
    inputs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:1a3df49b5875d10a79fd6645fb694e4896aa13df5e30447db4900604fc28dc8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - iMLGAM
  - qs
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# roc-auc-discrimination-assessment

## Summary

Scores the training (70% split), validation (30% split), and test ICI cohorts with the GA-fitted iMLGAM model and evaluates ROC-AUC discrimination of the resulting riskscores against the 'res' ICB response labels, confirming that lower iMLGAM scores mark responders.

## When to use

Use when a fitted ga_fit (GA_train output) exists and you must score train/validat/test prediction matrices and quantify ICB-response discrimination with ROC-AUC per cohort.

## When NOT to use

- You are still constructing base learners or selecting features with ABESS — scoring requires a ga_fit model from GA_train (Step 3 output), not the raw gene-pair training set
- The outcome labels ('res') are missing or misaligned — row names of prediction matrices must be intersected with the gene-pair cohort data before scoring, otherwise AUC is meaningless
- The task is prognostic (survival/time-to-event) evaluation rather than binary ICB responder/non-responder discrimination

## Inputs

- ga_fit: fitted genetic-algorithm model from GA_train(trainres) in the iMLGAM R package
- trainres / validatres / testres: prediction matrices from basic_learner_predictor with 'res' outcome labels (row names intersected with 'ICI cohort gene pairs.qs')
- 'Sample File.qs' and 'ICI cohort gene pairs.qs' files read with the qs package

## Outputs

- riskscore columns appended to trainres, validatres, and testres
- per-split ROC-AUC values (expected ~0.76 train, ~0.70 test, ~0.72 validation)
- combined table imlgam_risk_scores.csv

## How to apply

Load 'Sample File.qs' and 'ICI cohort gene pairs.qs' with the qs package, keep the 'res' outcome column, and intersect row names of each split's prediction matrix (trainres, validatres, testres) with the gene-pair matrices via `intersect(rownames(...), ...)` before cbind-ing the labels. Score each cohort by calling predict(ga_fit, ...) on trainres, validatres, and testres to append a 'riskscore' column, then compute ROC-AUC of riskscore against the 'res' labels for each split. Verify directionality: responders must show significantly lower riskscores, consistent with the reported AUCs of ~0.76 (train), ~0.70 (test), and ~0.72 (validation). Assemble the three scored cohorts into imlgam_risk_scores.csv. This step follows Step 4 of the iMLGAM workflow, which consumes the ga_fit model produced by GA_train in Step 3.

## Related tools

- **iMLGAM** (R package providing GA_train and predict() to fit the genetic-algorithm model and compute riskscores per cohort) — https://github.com/Yelab1994/iMLGAM
- **R** (runtime environment for the scoring and ROC-AUC evaluation workflow)
- **qs** (loads the 'Sample File.qs' and 'ICI cohort gene pairs.qs' cohort/prediction-matrix inputs)

## Examples

```
trainres$riskscore <- predict(ga_fit, trainres)
```

## Evaluation signals

- riskscore values are finite for all patients in each split (no NA/NaN after predict(ga_fit, ...))
- Per-split ROC-AUC reaches the reported discrimination levels: ~0.76 training, ~0.70 testing, ~0.72 validation
- Responders show significantly lower riskscores than non-responders in all three cohorts (reported all p < 0.001)
- Row-name intersection between prediction matrices and 'ICI cohort gene pairs.qs' labels yields matched samples before cbind-ing 'res'
- imlgam_risk_scores.csv contains all three scored cohorts (train, test, validation) with a riskscore column

## Limitations

- The 11 RNA-seq cohorts analyzed cover only six cancer types, so pan-cancer predictive efficacy requires further validation through prospective ICB trials
- No large-scale, multi-center pan-cancer cohort validation exists, limiting the model's generalizability
- Score directionality is inverted relative to naive risk interpretation: lower iMLGAM scores indicate greater likelihood of ICB benefit

## Evidence

- [results] The iMLGAM score discriminated ICB responders from non-responders across all three cohorts (lower scores indicated greater likelihood of benefit, all p < 0.001), with ROC-AUC values of 0.76 for the training set: [evidence span withheld — anchor status: not_found]
- [readme] Step 4 risk score code: "## Step 4: Calculating Risk Scores for Each set"
- [readme] ga_fit model fitting: "ga_fit <- GA_train(trainres)"
- [readme] predict call appending riskscore: "trainres $riskscore <- predict(ga_fit ,trainres)"
- [readme] row-name intersection with gene-pair labels: "jj <- intersect(rownames(dat$trainset_7),rownames(trainres))"
