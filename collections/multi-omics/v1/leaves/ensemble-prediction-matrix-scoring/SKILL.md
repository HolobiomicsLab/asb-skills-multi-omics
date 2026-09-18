---
name: ensemble-prediction-matrix-scoring
description: Use when base-learner prediction matrices (trainres/validatres/testres)
  and a GA_train-fitted model already exist and per-cohort risk scores with ROC-AUC
  discrimination must be computed for ICB response prediction.
when_to_use_negative:
- Input is raw gene-pair feature tables and base learners have not yet been constructed
  — the ABESS feature selection (support.size=0:100) and basic_learner() steps must
  run first
- No GA_train-fitted model (ga_fit) is available; predict() requires the ensemble
  fitted on the training prediction matrix
- The goal is de novo biomarker discovery or mechanism analysis (e.g. CRISPR/Cox-based
  CEP55 prioritization) rather than scoring existing cohorts with the iMLGAM signature
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3518
tools:
- name: iMLGAM
  role: R package supplying GA_train() to fit the genetic-algorithm ensemble and predict()
    to compute riskscore values per cohort
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Runtime environment in which the iMLGAM scoring and ROC-AUC evaluation are
    executed
- name: qs
  role: Loads the response-labelled splits ('Sample File.qs', 'ICI cohort gene pairs.qs')
    whose row names anchor prediction matrices to outcome labels
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:527063d57eb1d0d8dcd63673f13a48f971bed0ee50521767eac218cb890c4010
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - iMLGAM
  - qs
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 9
---

# ensemble-prediction-matrix-scoring

## Summary

Score ICB-response cohorts with the iMLGAM genetic-algorithm-fitted ensemble model (ga_fit) by calling predict() on base-learner prediction matrices, appending a 'riskscore' column to the training, testing, and validation sets for ROC-AUC discrimination.

## When to use

Use when base-learner prediction matrices (trainres/validatres/testres) and a GA_train-fitted model already exist and per-cohort risk scores with ROC-AUC discrimination must be computed for ICB response prediction.

## When NOT to use

- Input is raw gene-pair feature tables and base learners have not yet been constructed — the ABESS feature selection (support.size=0:100) and basic_learner() steps must run first
- No GA_train-fitted model (ga_fit) is available; predict() requires the ensemble fitted on the training prediction matrix
- The goal is de novo biomarker discovery or mechanism analysis (e.g. CRISPR/Cox-based CEP55 prioritization) rather than scoring existing cohorts with the iMLGAM signature

## Inputs

- Prediction matrices trainres, validatres, testres produced by basic_learner_predictor() on the 70% training split, 30% validation split, and test set
- Response labels ('res' column) from 'Sample File.qs' and 'ICI cohort gene pairs.qs' loaded via qs::qread, intersected by row names
- GA_train-fitted ensemble model object ga_fit from the upstream base-learner optimization step

## Outputs

- trainres, validatres, testres data frames each with an appended 'riskscore' column
- Per-split ROC-AUC discrimination values (expected ~0.76 train, ~0.70 test, ~0.72 validation)
- Combined table of the three scored cohorts (e.g. imlgam_risk_scores.csv)

## How to apply

Load the response-labelled data from 'Sample File.qs' and 'ICI cohort gene pairs.qs' with qs::qread, keep only the outcome column 'res' after dropping columns 1, 3, 4, and intersect row names of each split with the prediction matrices. After ga_fit has been fitted via GA_train(trainres) in the upstream genetic-algorithm optimization step, score each split by calling predict(ga_fit, ...) on trainres, validatres, and testres, appending a 'riskscore' column to each. Compare riskscores against the 'res' outcome labels to compute ROC-AUC for the train/validation/test sets; the article reports discrimination of ~0.76 (training), ~0.70 (testing), and ~0.72 (validation), all p < 0.001. Confirm directionality: lower riskscores indicate greater likelihood of ICB benefit, consistent with the low-score inflammatory immune phenotype (CD8+ T-cell infiltration, elevated PD-L1/PD-1/CTLA-4). Assemble the three scored cohorts (e.g. into imlgam_risk_scores.csv) for downstream biomarker evaluation.

## Related tools

- **iMLGAM** (R package supplying GA_train() to fit the genetic-algorithm ensemble and predict() to compute riskscore values per cohort) — https://github.com/Yelab1994/iMLGAM
- **R** (Runtime environment in which the iMLGAM scoring and ROC-AUC evaluation are executed)
- **qs** (Loads the response-labelled splits ('Sample File.qs', 'ICI cohort gene pairs.qs') whose row names anchor prediction matrices to outcome labels)

## Examples

```
library(iMLGAM); trainres$riskscore <- predict(ga_fit, trainres); validatres$riskscore <- predict(ga_fit, validatres); testres$riskscore <- predict(ga_fit, testres)
```

## Evaluation signals

- riskscore values are finite for all patients in each of the train/validation/test splits
- Per-split ROC-AUC reaches the reported discrimination levels: ~0.76 (training), ~0.70 (testing), ~0.72 (validation), with p < 0.001
- Responders show significantly lower riskscores than non-responders (score directionality check)
- Row names of each scored matrix match the intersection with the 'res'-labelled cohorts from the .qs files
- All three scored cohorts assemble into a single table (e.g. imlgam_risk_scores.csv) without missing outcome labels

## Limitations

- The 11 RNA-seq cohorts cover only six cancer types; pan-cancer predictive efficacy of the iMLGAM score requires validation in prospective ICB trials
- No large-scale, multi-center pan-cancer cohort validation, limiting the model's generalizability
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses

## Evidence

- [results] with ROC-AUC values of 0.76 for the training set, 0.70 for the testing set, and 0.72 for the validation set.: [evidence span withheld — anchor status: not_found]
- [methods] Score each set by calling predict(ga_fit, ...) on trainres, validatres, and testres to append a 'riskscore' column: "## Step 4: Calculating Risk Scores for Each set"
- [readme] Obtaining the evaluation score of the training set: "trainres $riskscore <- predict(ga_fit ,trainres)"
- [abstract] iMLGAM scores exhibit superior predictive performance across independent cohorts, with lower scores correlating with enhanced therapeutic responses: "iMLGAM scores exhibit superior predictive performance across independent cohorts, with lower scores correlating significantly with enhanced therapeutic responses"
