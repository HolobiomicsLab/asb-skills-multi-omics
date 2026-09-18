---
name: gene-pair-feature-selection
description: Use when building an ICB response prediction model from RNA-seq expression/gene-pair
  matrices and you need batch-robust feature reduction (e.g., hundreds of gene pairs
  down to a top-5 support set) prior to ensemble learning.
when_to_use_negative:
- Input is already a reduced feature table with only a handful of candidate predictors
  — best-subset scanning over support.size=0:100 is unnecessary
- Data are not gene-expression/gene-pair based (e.g., imaging or clinical covariates)
  — gene pairing's batch-effect rationale does not transfer
- The goal is the full iMLGAM score computation on an already-trained GA model — this
  skill covers only the upstream feature-selection step, not GA_train or predict(ga_fit,
  ...)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: abess
  role: Adaptive best-subset selection algorithm used to scan support.size=0:100 and
    extract the top 5 support gene-pair variables from the training set
- name: qs
  role: Loads the .qs cohort files (Sample File.qs) containing training, validation,
    and test gene-pair matrices
- name: iMLGAM
  role: R package whose pipeline consumes the ABESS-selected features for downstream
    basic_learner ensemble construction
  repo: https://github.com/Yelab1994/iMLGAM
- name: devtools
  role: Installs the iMLGAM package from GitHub before running the selection pipeline
  repo: https://github.com/Yelab1994/iMLGAM
merged_aliases:
- gene-pair-feature-selection-abess
merged_alias_records:
- alias: gene-pair-feature-selection-abess
  slug: gene-pair-feature-selection-abess
  jaccard_score: 0.8
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:5b9e8f5b7285ace26f4c2c96af8c420f7c2667358435bb6b9da562586d6b8cc8
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# gene-pair-feature-selection

## Summary

Selects a compact set of predictive gene-pair features from RNA-seq ICB cohort data using the ABESS adaptive best-subset selection algorithm, reducing batch effects via gene pairing while narrowing hundreds of features to the top 5 support variables before ensemble model construction.

## When to use

Use when building an ICB response prediction model from RNA-seq expression/gene-pair matrices and you need batch-robust feature reduction (e.g., hundreds of gene pairs down to a top-5 support set) prior to ensemble learning.

## When NOT to use

- Input is already a reduced feature table with only a handful of candidate predictors — best-subset scanning over support.size=0:100 is unnecessary
- Data are not gene-expression/gene-pair based (e.g., imaging or clinical covariates) — gene pairing's batch-effect rationale does not transfer
- The goal is the full iMLGAM score computation on an already-trained GA model — this skill covers only the upstream feature-selection step, not GA_train or predict(ga_fit, ...)

## Inputs

- Sample File.qs cohort file (qs format) containing trainset_7, trainset_3, and testset expression/gene-pair matrices with 'res' response column
- Training-set feature matrix (gene-pair columns) and binary/continuous response labels ('res')

## Outputs

- abess_fit model object from abess(support.size=0:100)
- key_gene: character vector of 5 selected gene-pair features from extract(abess_fit, 5)
- Subsetted trainset, validationset, and testset data frames containing only 'res' plus the 5 selected gene-pair columns

## How to apply

Load the cohort file (Sample File.qs) with qs::qread and split into the 70% training set (trainset_7), 30% validation set (trainset_3), and test set, dropping the first, third, and fourth metadata columns and renaming column 1 to 'res'. Remove training rows with NA response labels before selection. Run abess() on the training feature matrix (all columns except 'res') with the response as outcome and support.size=0:100 to scan model sizes, then extract the 5 top-ranked support variables via extract(abess_fit, 5)[4][1]$support.vars. Subset training, validation, and test sets to the 'res' column plus the selected key_gene gene pairs so downstream base learners (basic_learner with CVnumber=10, Cvrepeats=5, ncore=8) train only on these batch-robust features. Feature selection must be fit on the training set only to avoid information leakage into validation/test.

## Related tools

- **abess** (Adaptive best-subset selection algorithm used to scan support.size=0:100 and extract the top 5 support gene-pair variables from the training set)
- **qs** (Loads the .qs cohort files (Sample File.qs) containing training, validation, and test gene-pair matrices)
- **iMLGAM** (R package whose pipeline consumes the ABESS-selected features for downstream basic_learner ensemble construction) — https://github.com/Yelab1994/iMLGAM
- **devtools** (Installs the iMLGAM package from GitHub before running the selection pipeline) — https://github.com/Yelab1994/iMLGAM

## Examples

```
abess_fit <- abess(trainset[,-1], trainset$res, support.size=0:100); key_gene <- extract(abess_fit, 5)[4][1]$support.vars
```

## Evaluation signals

- extract(abess_fit, 5) returns exactly 5 support variables (key_gene) drawn from the training feature matrix
- Subsetted trainset, validationset, and testset all contain only the 'res' column plus the identical 5 key_gene columns
- Training rows with NA 'res' values were removed before abess() was fit (trainset[!is.na(trainset$res),])
- The reduced feature set supports downstream 10-fold CV base-learner construction (basic_learner with CVnumber=10, Cvrepeats=5, ncore=8) without errors
- ABESS was fit only on the training set; validation and test sets were subset but never used for selection

## Limitations

- The top-5 support size is fixed by the pipeline recipe (extract(abess_fit, 5)); other support sizes are scanned (0:100) but only 5 variables are retained for base-learner construction
- The 11 RNA-seq cohorts analyzed cover only six cancer types, and some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses
- Gene-pair features reduce batch effects but the resulting model's pan-cancer predictive efficacy still requires validation in prospective ICB trials

## Evidence

- [readme] abess_fit <- abess(trainset[,-1], trainset$res,support.size=0:100): "abess_fit <- abess(trainset[,-1], trainset$res,support.size=0:100)"
- [readme] key_gene extraction: "key_gene <- extract(abess_fit, 5)[4][1]$support.vars"
- [methods] NA response removal before selection: "Feature Selection using ABESS Algorithm
trainset <- trainset[!is.na(trainset$res),]"
- [intro] gene-pairing rationale for batch-effect reduction: [evidence span withheld — anchor status: not_found]
- [methods] support.size scan across 0:100: [evidence span withheld — anchor status: not_found]
