---
name: cohort-stratification-train-validation-test
description: Use when evaluating an ICB response predictor that must be benchmarked
  against published signatures across independent training, validation, and test ICB
  RNA-seq cohorts with a 'res' outcome column.
when_to_use_negative:
- Input is a single-cohort study without independent validation/test ICB cohorts —
  the skill's premise is three-way stratification and cross-cohort benchmarking
- The goal is de novo signature discovery or feature selection (use the ABESS/basic_learner
  steps instead) rather than scoring and benchmarking on pre-split cohorts
- Cohorts lack the 'res' immunotherapy-response outcome column needed for AUC evaluation
tools:
- name: iMLGAM
  role: R package whose GA-fit model (ga_fit) produces the riskscore for each stratified
    cohort via predict()
  repo: https://github.com/Yelab1994/iMLGAM
- name: qs
  role: Reads the .qs cohort files ('Sample File.qs', 'ICI cohort gene pairs.qs')
    into R via qread()
- name: iMLAGA
  role: Genetic-learning algorithm package used to obtain the optimal base-learner
    combination before cohort scoring
  repo: https://github.com/Yelab1994/iMLAGA
- name: R
  role: Environment in which cohorts are stratified, scored, and the 12 published
    signatures are computed and compared
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:ba40bc8f6817d4f562487ddfe880d685084f63ae102664c1d290965b52d818c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
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

# cohort-stratification-train-validation-test

## Summary

Splits pan-cancer ICB cohort data into training (trainset_7), validation (trainset_3), and test (testset) cohorts loaded from 'Sample File.qs' and 'ICI cohort gene pairs.qs', then scores each with the fitted genetic-algorithm ensemble via predict(ga_fit, cohort) so that iMLGAM riskscores can be benchmarked against 12 published immunotherapy signatures (CAF, IFNγ, chemokines, Davoli_IS, CEP, CYT, TLS, TIDE, NLPR3, Roh_IS, Ayers_expIS, IPS).

## When to use

Use when evaluating an ICB response predictor that must be benchmarked against published signatures across independent training, validation, and test ICB RNA-seq cohorts with a 'res' outcome column.

## When NOT to use

- Input is a single-cohort study without independent validation/test ICB cohorts — the skill's premise is three-way stratification and cross-cohort benchmarking
- The goal is de novo signature discovery or feature selection (use the ABESS/basic_learner steps instead) rather than scoring and benchmarking on pre-split cohorts
- Cohorts lack the 'res' immunotherapy-response outcome column needed for AUC evaluation

## Inputs

- Sample File.qs — cohort list with trainset_7, trainset_3, and testset expression/prediction matrices
- ICI cohort gene pairs.qs — cohort outcomes ('res' column) aligned by row names
- Fitted genetic-algorithm model object returned by GA_train(trainres)

## Outputs

- Per-cohort iMLGAM riskscore columns for training, validation, and test sets
- Per-cohort AUC values for iMLGAM and each of the 12 published signatures
- signature_comparison_table.csv contrasting iMLGAM against each published signature across all three cohorts

## How to apply

Load the cohort prediction-matrix files with qs::qread from 'Sample File.qs' and 'ICI cohort gene pairs.qs', assigning dat$trainset_7 as the training set, dat$trainset_3 as the validation set, and dat$testset as the test set; for each cohort drop the first, third, and fourth columns and rename the first column 'res' (the outcome). After the ABESS feature-selection and GA training steps (GA_train on the base-learner prediction matrices merged with cohort outcomes), score each cohort with predict(ga_fit, cohort) to append the iMLGAM riskscore column. Compute the 12 published immune signatures (CAF, IFNγ, chemokines, Davoli_IS, CEP, CYT, TLS, TIDE, NLPR3, Roh_IS, Ayers_expIS, IPS) from the same RNA-seq expression matrices, then compare AUC for immunotherapy response of each signature versus the iMLGAM riskscore per cohort using the 'res' column. Assemble the results into signature_comparison_table.csv; correct application shows iMLGAM consistently outperforming all 12 published signatures across training, testing, and validation cohorts.

## Related tools

- **iMLGAM** (R package whose GA-fit model (ga_fit) produces the riskscore for each stratified cohort via predict()) — https://github.com/Yelab1994/iMLGAM
- **qs** (Reads the .qs cohort files ('Sample File.qs', 'ICI cohort gene pairs.qs') into R via qread())
- **iMLAGA** (Genetic-learning algorithm package used to obtain the optimal base-learner combination before cohort scoring) — https://github.com/Yelab1994/iMLAGA
- **R** (Environment in which cohorts are stratified, scored, and the 12 published signatures are computed and compared)

## Examples

```
dat <- qread("Sample File.qs"); trainset <- dat$trainset_7; validationset <- dat$trainset_3; testset <- dat$testset
```

## Evaluation signals

- signature_comparison_table.csv shows the iMLGAM riskscore outperforming all 12 published signatures (CAF, IFNγ, chemokines, Davoli_IS, CEP, CYT, TLS, TIDE, NLPR3, Roh_IS, Ayers_expIS, IPS) in predictive performance across training, testing, and validation cohorts
- Row names of trainres, validatres, and testres match the intersect with dat$trainset_7, dat$trainset_3, and dat$testset row names from 'ICI cohort gene pairs.qs' after cbind of the 'res' outcome
- Each cohort object has a non-NA 'res' column (trainset is filtered with trainset[!is.na(trainset$res),]) and a populated riskscore column after predict(ga_fit, cohort)
- Lower iMLGAM riskscores correlate with enhanced therapeutic responses, consistent with the reported direction of association

## Limitations

- The 11 RNA-seq cohorts cover only six cancer types, so pan-cancer predictive efficacy of the iMLGAM score requires further validation through prospective ICB trials
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses
- No large-scale, multi-center pan-cancer cohort validation exists, limiting the model's generalizability

## Evidence

- [readme] cohort assignment from README: "Determining the training set
trainset <- dat$trainset_7
#Determining the validation set
validationset <- dat$trainset_3"
- [readme] cohort scoring from README: "testres$riskscore<- predict(ga_fit ,testres)"
- [abstract] finding being validated: "iMLGAM scores exhibit superior predictive performance across independent cohorts, with lower scores correlating significantly with enhanced therapeutic responses"
- [readme] outcome column definition: "#The first column is the outcome (with row names as 'res'), and the subsequent columns are features."
