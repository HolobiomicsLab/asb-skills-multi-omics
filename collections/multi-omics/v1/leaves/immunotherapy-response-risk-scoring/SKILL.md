---
name: immunotherapy-response-risk-scoring
description: Use when building a predictive risk score for immunotherapy (ICB) response
  from RNA-seq gene-pair features, needing ensemble ML with automated base-learner
  optimization via a genetic algorithm.
when_to_use_negative:
- Input is a single-omics feature table without gene-pair (relative expression ordering)
  features — gene pairing is used specifically to reduce batch effects
- The goal is biomarker discovery or mechanistic analysis (e.g. CEP55 CRISPR validation)
  rather than building a response-prediction risk score
- Cohorts lack response labels ('res') or clinical annotations for ICB outcome, since
  the pipeline trains on the 'res' outcome column
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: iMLGAM
  role: R package providing basic_learner(), basic_learner_predictor(), GA_train(),
    and predict() for ensemble construction and risk scoring
  repo: https://github.com/Yelab1994/iMLGAM
- name: abess
  role: Adaptive best-subset selection (abess() with support.size=0:100) to select
    the top 5 gene-pair features
- name: qs
  role: Fast serialization used to load 'Sample File.qs' and 'ICI cohort gene pairs.qs'
    cohort files
- name: devtools
  role: Used to install the iMLGAM package from GitHub via install_github()
  repo: https://github.com/Yelab1994/iMLGAM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:9cfa0766f0fb4d6bba9a444ca771bfb2aed80942ca7409af4a3a00edbae050de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - abess
  - devtools
  - iMLGAM
  - qs
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 9
---

# immunotherapy-response-risk-scoring

## Summary

Reconstructs the iMLGAM risk-scoring pipeline: ABESS feature selection on gene pairs, an ensemble of cross-validated base learners, genetic-algorithm (GA) selection of the optimal learner combination, and stepwise-regression risk scores for ICB response prediction in pan-cancer cohorts.

## When to use

Use when building a predictive risk score for immunotherapy (ICB) response from RNA-seq gene-pair features, needing ensemble ML with automated base-learner optimization via a genetic algorithm.

## When NOT to use

- Input is a single-omics feature table without gene-pair (relative expression ordering) features — gene pairing is used specifically to reduce batch effects
- The goal is biomarker discovery or mechanistic analysis (e.g. CEP55 CRISPR validation) rather than building a response-prediction risk score
- Cohorts lack response labels ('res') or clinical annotations for ICB outcome, since the pipeline trains on the 'res' outcome column

## Inputs

- Sample File.qs (list of trainset_7, trainset_3, testset data frames; first column is the outcome 'res', remaining columns are gene-pair features)
- ICI cohort gene pairs.qs (cohort outcomes with row names matching prediction-matrix samples)

## Outputs

- abess_fit object and key_gene list of 5 top-ranked gene pairs
- basic_learner_list of trained base-learner models
- trainres, validatres, testres prediction matrices with merged 'res' outcomes
- ga_fit GA-optimized ensemble model
- Numeric riskscore column for every sample in training, validation, and test sets

## How to apply

Load the example cohort with qs::qread('Sample File.qs') and split into trainset_7 (70% training), trainset_3 (validation), and testset, removing training rows with missing 'res' labels. Run ABESS adaptive best-subset selection with support.size=0:100 on the training feature matrix, extract the top 5 gene pairs via extract(abess_fit, 5), and subset all cohorts to those features. Build base learners with basic_learner() using 10-fold cross-validation, 5 CV repeats, and 8 cores, then generate per-model prediction matrices for each cohort with basic_learner_predictor(). Merge the response outcomes from 'ICI cohort gene pairs.qs' onto the prediction matrices by intersecting row names, run GA_train() on the training prediction matrix to select the optimal base-learner combination, and apply predict(ga_fit, ...) to each cohort to append the final 'riskscore' column. Lower iMLGAM scores indicate enhanced ICB therapeutic response.

## Related tools

- **iMLGAM** (R package providing basic_learner(), basic_learner_predictor(), GA_train(), and predict() for ensemble construction and risk scoring) — https://github.com/Yelab1994/iMLGAM
- **abess** (Adaptive best-subset selection (abess() with support.size=0:100) to select the top 5 gene-pair features)
- **qs** (Fast serialization used to load 'Sample File.qs' and 'ICI cohort gene pairs.qs' cohort files)
- **devtools** (Used to install the iMLGAM package from GitHub via install_github()) — https://github.com/Yelab1994/iMLGAM

## Examples

```
library(iMLGAM); basic_learner_list <- basic_learner(trainingset=trainset, CVnumber=10, Cvrepeats=5, ncore=8); ga_fit <- GA_train(trainres); trainres$riskscore <- predict(ga_fit, trainres)
```

## Evaluation signals

- Pipeline completes through GA_train convergence without error
- predict(ga_fit, ...) yields a numeric riskscore for every sample in trainres, validatres, and testres
- ABESS extract(abess_fit, 5) returns exactly 5 gene-pair features (key_gene) shared across all three subsetted cohorts
- Prediction matrices from basic_learner_predictor() successfully intersect by row names with the 'ICI cohort gene pairs.qs' outcomes before cbind
- Lower computed iMLGAM scores correspond to enhanced ICB therapeutic response, per the paper's score convention

## Limitations

- The 11 RNA-seq cohorts analyzed represent only six cancer types; pan-cancer predictive efficacy requires validation in prospective ICB trials
- No large-scale, multi-center pan-cancer cohort validation, limiting generalizability
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses
- Ensemble learning carries risks of overfitting and must handle heterogeneity of sequencing platforms

## Evidence

- [readme] abess_fit <- abess(trainset[,-1], trainset$res,support.size=0:100): "abess_fit <- abess(trainset[,-1], trainset$res,support.size=0:100)"
- [readme] basic_learner_list <- basic_learner(trainingset=trainset , CVnumber=10, Cvrepeats=5, ncore=8): "basic_learner_list <- basic_learner(trainingset=trainset , CVnumber=10, Cvrepeats=5, ncore=8)"
- [abstract] GA ensemble construction: "iMLGAM scores exhibit superior predictive performance across independent cohorts, with lower scores correlating significantly with enhanced therapeutic responses"
- [methods] GA selection of base learners: "## Step 3: Running a genetic learning algorithm to obtain the optimal combination of base learners"
- [intro] ensemble rationale: "ensemble learning methodologies have been demonstrated to provide enhanced predictive accuracy for outcomes when compared to individual machine learning approaches"
