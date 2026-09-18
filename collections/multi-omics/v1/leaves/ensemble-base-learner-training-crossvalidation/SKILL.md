---
name: ensemble-base-learner-training-crossvalidation
description: Use when a gene-pair feature table with a binary ICB response column
  ('res') must be converted into an ensemble of cross-validated base-learner predictions,
  as a prerequisite for GA-based learner combination and riskscore computation.
when_to_use_negative:
- The data are not RNA-seq gene-pair features with a defined binary 'res' ICB-response
  column — the gene-pairing strategy is designed to reduce batch effects in transcriptomic
  data
- Only a single, already-validated model is needed and no learner combination is planned
  — the ensemble and GA step would add overfitting risk without benefit
- The cohort is too small to support 10-fold cross-validation with 5 repeats per base
  learner (e.g., the GBM cohort, where limited sample size yielded no significant
  responder/non-responder difference)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: iMLGAM
  role: R package supplying the basic_learner and basic_learner_predictor functions
    used to build and apply the cross-validated ensemble of base learners
  repo: https://github.com/Yelab1994/iMLGAM
- name: iMLAGA
  role: Dependency package installed from GitHub, required for the genetic-algorithm
    learner combination (GA_train) that consumes the ensemble predictions
  repo: https://github.com/Yelab1994/iMLAGA
- name: abess
  role: Feature selection (support.size=0:100, extract 5 support variables) that defines
    the reduced trainset passed to basic_learner
- name: qs
  role: Loads the .qs sample-expression and ICI-cohort outcome files that form the
    training/validation/test inputs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:11114b07743c65c02d593af8e4eac818b420d2737cd4834a01099dc5520d5418
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - abess
  - devtools
  - iMLAGA
  - iMLGAM
  - qs
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.1111111111111111
n_claims: 9
---

# ensemble-base-learner-training-crossvalidation

## Summary

Trains an ensemble of machine learning base learners on gene-pair transcriptomic features (pre-selected by ABESS) with repeated 10-fold cross-validation, then produces per-set prediction matrices that feed the genetic-algorithm combination step of the iMLGAM ICB-response scoring workflow.

## When to use

Use when a gene-pair feature table with a binary ICB response column ('res') must be converted into an ensemble of cross-validated base-learner predictions, as a prerequisite for GA-based learner combination and riskscore computation.

## When NOT to use

- The data are not RNA-seq gene-pair features with a defined binary 'res' ICB-response column — the gene-pairing strategy is designed to reduce batch effects in transcriptomic data
- Only a single, already-validated model is needed and no learner combination is planned — the ensemble and GA step would add overfitting risk without benefit
- The cohort is too small to support 10-fold cross-validation with 5 repeats per base learner (e.g., the GBM cohort, where limited sample size yielded no significant responder/non-responder difference)

## Inputs

- 'Sample File.qs' gene-pair expression matrices (loaded via qs::qread) partitioned into trainset_7, trainset_3 and testset, each with a binary 'res' outcome column
- 'ICI cohort gene pairs.qs' file providing outcome annotations aligned by sample row names
- key_gene list of 5 variables extracted from the ABESS fit

## Outputs

- basic_learner_list: fitted ensemble of base learners trained with 10-fold CV repeated 5 times
- Prediction matrices trainres, validatres, testres (one column per base learner, merged with 'res')
- ga_fit: genetic-algorithm-optimized combination of base learners
- Per-sample riskscore values for training, validation and test sets

## How to apply

First reduce dimensionality with the ABESS algorithm (abess(trainset[,-1], trainset$res, support.size=0:100)) and keep the 5 support variables (key_gene), subsetting trainset, validationset and testset to 'res' plus key_gene. Then call basic_learner(trainingset=trainset, CVnumber=10, Cvrepeats=5, ncore=8) to build base learners under 10-fold cross-validation repeated 5 times on 8 cores. Apply basic_learner_predictor to each set to obtain prediction matrices, then merge each matrix with ICI cohort outcome annotations by intersecting row names (e.g., jj <- intersect(rownames(dat$trainset_7), rownames(trainres))). Finally run GA_train(trainres) to obtain the genetic-algorithm-optimized learner combination and score all sets with predict(ga_fit, ...). The rationale is that ensemble learning outperforms single models on transcriptomic data, while repeated CV guards against overfitting when learners are combined.

## Related tools

- **iMLGAM** (R package supplying the basic_learner and basic_learner_predictor functions used to build and apply the cross-validated ensemble of base learners) — https://github.com/Yelab1994/iMLGAM
- **iMLAGA** (Dependency package installed from GitHub, required for the genetic-algorithm learner combination (GA_train) that consumes the ensemble predictions) — https://github.com/Yelab1994/iMLAGA
- **abess** (Feature selection (support.size=0:100, extract 5 support variables) that defines the reduced trainset passed to basic_learner)
- **qs** (Loads the .qs sample-expression and ICI-cohort outcome files that form the training/validation/test inputs)

## Examples

```
library(iMLGAM); basic_learner_list <- basic_learner(trainingset=trainset, CVnumber=10, Cvrepeats=5, ncore=8)
```

## Evaluation signals

- ABESS feature selection returns exactly 5 support variables (key_gene) via extract(abess_fit, 5); trainset, validationset and testset are all reduced to 'res' + key_gene before model building
- basic_learner runs with CVnumber=10 and Cvrepeats=5 on ncore=8 — the cross-validation protocol used in the paper
- Row-name intersections with the ICI cohort files are non-empty for all three sets (jj <- intersect(rownames(dat$trainset_7), rownames(trainres))) so predictions align with outcomes
- riskscore is computable via predict(ga_fit, ...) for trainres, validatres and testres without error
- Downstream stratification by tumor type yields ROC-AUC > 0.6 with lower scores in responders for RCC, urothelial carcinoma, melanoma and gastric cancer (p < 0.05)

## Limitations

- Ensemble learning carries a risk of overfitting and faces heterogeneity of sequencing platforms, with no automated protocols for basic-learner identification acknowledged in the paper
- The 11 RNA-seq cohorts cover only six cancer types; the model's pan-cancer predictive efficacy requires further validation in prospective ICB trials and large-scale multi-center pan-cancer cohorts
- Datasets lacking critical clinical annotations can hinder comprehensive biomarker analyses of the resulting scores

## Evidence

- [readme] basic_learner_list <- basic_learner(trainingset=trainset , CVnumber=10, Cvrepeats=5, ncore=8): "basic_learner_list <- basic_learner(trainingset=trainset , CVnumber=10, Cvrepeats=5, ncore=8)"
- [intro] Ensemble learning provides enhanced predictive accuracy compared to individual ML approaches: "ensemble learning methodologies have been demonstrated to provide enhanced predictive accuracy for outcomes when compared to individual machine learning approaches"
- [readme] Step 2 constructs prediction matrices for all base learners across train/validation/test sets: "## Step 2: Constructing prediction matrices for various machine learning models"
- [methods] abess feature selection with support.size=0:100: "abess_fit <- abess(trainset[,-1], trainset$res,support.size=0:100)"
