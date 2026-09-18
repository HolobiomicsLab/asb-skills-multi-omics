---
name: cohort-data-splitting
description: Use when an RNA-seq immunotherapy cohort arrives as a multi-list .qs
  file and a reproducible train/validation/test split with cleaned 'res' outcome labels
  is needed before ensemble modeling.
when_to_use_negative:
- Input is already a single modeling matrix or feature table with no trainset_7/trainset_3/testset
  list structure to split
- The goal is unsupervised or prognostic-only analysis where no 'res' ICB response
  labels exist to drive the split and NA-filtering
- A single-cohort cross-validation design is required without an independent held-out
  test set
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: qs
  role: Loads the serialized .qs cohort file (Sample File.qs, ICI cohort gene pairs.qs)
    into R for splitting
- name: abess
  role: Consumes the cleaned training split for adaptive best-subset feature selection
    (support.size=0:100) after NA-res rows are removed
- name: iMLGAM
  role: R package whose basic_learner, basic_learner_predictor, GA_train, and predict
    functions operate on the three split cohorts
  repo: https://github.com/Yelab1994/iMLGAM
- name: devtools
  role: Installs the iMLGAM package from GitHub prior to running the split-and-model
    pipeline
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Execution environment for the cohort-splitting and downstream iMLGAM workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:38bbf32f5c05cef070738504143209f8c56990c9a93afb7f36ca16a8c04a7a63
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.1111111111111111
n_claims: 9
---

# cohort-data-splitting

## Summary

Partitions an ICB immunotherapy cohort (stored as a .qs file with trainset_7, trainset_3, and testset lists) into 70% training, 30% validation, and independent test sets, and removes training rows with missing response labels ('res') before downstream ABESS feature selection and base-learner construction in the iMLGAM pipeline.

## When to use

Use when an RNA-seq immunotherapy cohort arrives as a multi-list .qs file and a reproducible train/validation/test split with cleaned 'res' outcome labels is needed before ensemble modeling.

## When NOT to use

- Input is already a single modeling matrix or feature table with no trainset_7/trainset_3/testset list structure to split
- The goal is unsupervised or prognostic-only analysis where no 'res' ICB response labels exist to drive the split and NA-filtering
- A single-cohort cross-validation design is required without an independent held-out test set

## Inputs

- Cohort .qs file (e.g. 'Sample File.qs') containing named lists trainset_7, trainset_3, and testset with outcome column 'res' plus gene-pair feature columns
- 'ICI cohort gene pairs.qs' used later to re-merge response outcomes onto prediction matrices by intersecting row names

## Outputs

- trainset: data.frame with 'res' in column 1, gene-pair features thereafter, NA-res rows removed
- validationset: data.frame restricted to the same 'res' + selected feature columns
- testset: data.frame restricted to the same 'res' + selected feature columns

## How to apply

Load the cohort with qs::qread (e.g. 'Sample File.qs'), then in a loop strip columns 1, 3, and 4 from each list element and rename the first remaining column to 'res' so that the first column is the outcome and subsequent columns are gene-pair features. Assign dat$trainset_7 to the training set, dat$trainset_3 to the validation set, and dat$testset to the test set, yielding an approximately 70/30 training-validation partition plus an independent test set. Drop training rows where is.na(trainset$res) before feature selection, since ABESS (abess() with support.size=0:100) cannot fit on missing outcomes. Keep validation and test sets aligned to the same feature columns ('res' plus the ABESS-selected key_gene pairs) so downstream basic_learner(), basic_learner_predictor(), and GA_train()/predict(ga_fit, ...) apply identically across all three cohorts.

## Related tools

- **qs** (Loads the serialized .qs cohort file (Sample File.qs, ICI cohort gene pairs.qs) into R for splitting)
- **abess** (Consumes the cleaned training split for adaptive best-subset feature selection (support.size=0:100) after NA-res rows are removed)
- **iMLGAM** (R package whose basic_learner, basic_learner_predictor, GA_train, and predict functions operate on the three split cohorts) — https://github.com/Yelab1994/iMLGAM
- **devtools** (Installs the iMLGAM package from GitHub prior to running the split-and-model pipeline) — https://github.com/Yelab1994/iMLGAM
- **R** (Execution environment for the cohort-splitting and downstream iMLGAM workflow)

## Examples

```
library(qs); dat <- qread("Sample File.qs"); for (i in names(dat)) { dat[[i]] <- dat[[i]][,-c(1,3,4)]; colnames(dat[[i]])[1] <- "res" }; trainset <- dat$trainset_7; validationset <- dat$trainset_3; testset <- dat$testset
```

## Evaluation signals

- All three list elements (trainset_7, trainset_3, testset) contain a 'res' column in position 1 after the -c(1,3,4) column strip and rename loop
- sum(is.na(trainset$res)) == 0 before abess() is called on trainset[,-1]
- Validation and test data.frames have identical column names to trainset (c('res', key_gene)) after the key_gene subsetting step
- Row-name intersection against 'ICI cohort gene pairs.qs' (intersect(rownames(...))) yields non-empty matches so cbind(res=..., ...) merges outcomes onto trainres/validatres/testres
- Downstream predict(ga_fit, ...) produces a numeric 'riskscore' for every sample in all three cohorts

## Limitations

- Rows with missing response labels ('res') must be removed from the training set or ABESS fitting fails; validation/test handling of NAs is not explicitly covered
- The 11 RNA-seq cohorts analyzed cover only six cancer types, so pan-cancer generalizability of scores built on these splits requires prospective ICB trial validation
- Some datasets lack critical clinical annotations, which can hinder downstream biomarker analyses on the split cohorts

## Evidence

- [readme] split-and-assign from README: "#Determining the training set
trainset <- dat$trainset_7
#Determining the validation set
validationset <- dat$trainset_3"
- [readme] column cleanup loop from README: "dat[[i]] <- dat[[i]][,-c(1,3,4)]
  colnames( dat[[i]])[1] <- "res""
- [readme] NA filtering from README: "trainset <- trainset[!is.na(trainset$res),]"
- [readme] outcome/feature structure statement from README: "#The first column is the outcome (with row names as 'res'), and the subsequent columns are features."
- [methods] package loading from article methods: "dat <- qread("ICI cohort gene pairs.qs")"
