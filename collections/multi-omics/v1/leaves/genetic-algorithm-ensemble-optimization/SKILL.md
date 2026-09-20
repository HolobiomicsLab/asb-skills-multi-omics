---
name: genetic-algorithm-ensemble-optimization
description: Use when base-learner prediction matrices already exist (from basic_learner/basic_learner_predictor)
  and you must select the optimal combination of models and compute a final ensemble
  risk score for ICB response prediction.
when_to_use_negative:
- Input feature matrix has not yet been reduced by ABESS feature selection and base
  learners have not been built — run abess() and basic_learner()/basic_learner_predictor()
  first
- The goal is single-model prediction rather than an optimized stacking ensemble of
  machine-learning base learners
- Data lacks the 'res' response labels needed to merge onto the prediction matrices
  by intersecting row names
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3577
tools:
- name: iMLGAM
  role: R package providing GA_train() and the predict method for the GA-optimized
    ensemble scoring of base-learner prediction matrices
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Execution environment in which the iMLGAM package and GA-based ensemble optimization
    are run
- name: qs
  role: Loads the 'ICI cohort gene pairs.qs' file supplying response labels merged
    onto prediction matrices by intersecting row names
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:8ded6c6773bfabeb6c7fe6f9a37465a33741fcea46bfe1302a183405168cab89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - devtools
  - iMLGAM
  - qs
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# genetic-algorithm-ensemble-optimization

## Summary

Optimizes a stacking ensemble of machine-learned base learners with a genetic algorithm (GA_train) inside the iMLGAM R package, then applies predict(ga_fit, ...) to generate a numeric iMLGAM riskscore for training, validation, and test cohorts; lower scores indicate enhanced ICB therapeutic response.

## When to use

Use when base-learner prediction matrices already exist (from basic_learner/basic_learner_predictor) and you must select the optimal combination of models and compute a final ensemble risk score for ICB response prediction.

## When NOT to use

- Input feature matrix has not yet been reduced by ABESS feature selection and base learners have not been built — run abess() and basic_learner()/basic_learner_predictor() first
- The goal is single-model prediction rather than an optimized stacking ensemble of machine-learning base learners
- Data lacks the 'res' response labels needed to merge onto the prediction matrices by intersecting row names

## Inputs

- Training-set prediction matrix of per-base-learner predicted values (trainres) with response column 'res' merged from 'ICI cohort gene pairs.qs'
- Validation-set prediction matrix (validatres) with merged 'res' labels
- Test-set prediction matrix (testres) with merged 'res' labels

## Outputs

- ga_fit: trained GA-optimized ensemble model object
- Numeric 'riskscore' column appended to trainres, validatres, and testres

## How to apply

After building base learners via basic_learner() (CVnumber=10, Cvrepeats=5, ncore=8) and generating per-cohort prediction matrices with basic_learner_predictor(), merge the response labels ('res') from 'ICI cohort gene pairs.qs' onto each matrix by intersecting row names. Run GA_train(trainres) on the training prediction matrix to evolve the optimal subset of base learners (the paper's optimized ensemble retained ten models: m3, m5, m6, m16, m19, m23, m24, m29, m40, m46). Apply predict(ga_fit, ...) to the training, validation, and test matrices to append a numeric 'riskscore' column. The final score combines the selected learners' predicted values via stepwise logistic regression fitted in the training set. Under the paper's convention, lower iMLGAM scores correlate with enhanced ICB therapeutic response, so score direction must be interpreted accordingly.

## Related tools

- **iMLGAM** (R package providing GA_train() and the predict method for the GA-optimized ensemble scoring of base-learner prediction matrices) — https://github.com/Yelab1994/iMLGAM
- **R** (Execution environment in which the iMLGAM package and GA-based ensemble optimization are run)
- **qs** (Loads the 'ICI cohort gene pairs.qs' file supplying response labels merged onto prediction matrices by intersecting row names)

## Examples

```
library(iMLGAM); ga_fit <- GA_train(trainres); trainres$riskscore <- predict(ga_fit, trainres)
```

## Evaluation signals

- GA_train converges and returns a ga_fit object without error on the training prediction matrix
- predict(ga_fit, ...) yields a numeric 'riskscore' for every sample in the training, validation, and test sets
- Response labels 'res' are correctly merged by intersect(rownames(dat$...), rownames(...)) before GA_train
- Score direction matches the paper's convention: lower iMLGAM scores correlate with enhanced ICB therapeutic response
- The optimized ensemble corresponds to a selected subset of base learners (paper's optimum: ten models m3, m5, m6, m16, m19, m23, m24, m29, m40, m46)

## Limitations

- The 11 RNA-seq cohorts analyzed represent only six cancer types; pan-cancer predictive efficacy of the iMLGAM score requires further validation through prospective ICB trials
- No large-scale, multi-center pan-cancer cohort validation currently exists, limiting the model's generalizability
- Ensemble learning carries a risk of overfitting and must handle heterogeneity of sequencing platforms

## Evidence

- [readme] ga_fit <- GA_train(trainres): "#Merge with the prediction matrix
ga_fit <- GA_train(trainres)"
- [results] A genetic algorithm (GA) was implemented to enhance the ensemble model's accuracy; the optimized iMLGAM score incorporated ten selected base-learner models: [evidence span withheld — anchor status: not_found]
- [abstract] iMLGAM scores exhibit superior predictive performance across independent cohorts, with lower scores correlating significantly with enhanced therapeutic responses: "iMLGAM scores exhibit superior predictive performance across independent cohorts, with lower scores correlating significantly with enhanced therapeutic responses and outperforming existing clinical"
- [intro] several technical challenges have been identified, including the risk of overfitting, handling of the heterogeneity of sequencing platforms: "several technical challenges have been identified, including the risk of overfitting, handling of the heterogeneity of sequencing platforms, and the lack of automated protocols for the identification"
- [discussion] the 11 RNA‐seq cohorts analyzed represent a limited range of six cancer types, with the iMLGAM score's pan‐cancer predictive efficacy requiring further validation: "the 11 RNA‐seq cohorts analyzed represent a limited range of six cancer types, with the iMLGAM score's pan‐cancer predictive efficacy requiring further validation through prospective ICB trials"
