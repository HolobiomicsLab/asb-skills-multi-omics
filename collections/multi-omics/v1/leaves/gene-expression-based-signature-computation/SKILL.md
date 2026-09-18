---
name: gene-expression-based-signature-computation
description: Use when you have RNA-seq expression matrices from ICB-treated cohorts
  with a 'res' response column and need to compute/benchmark published immune signatures
  or compare a new score's AUC against them.
when_to_use_negative:
- The input is already a base-learner prediction matrix (output of basic_learner_predictor)
  rather than a raw RNA-seq expression matrix — signature computation from expression
  is unnecessary there
- The analysis goal is CRISPR-screen gene prioritization or CEP55 mechanism work,
  not signature computation or ICB-response benchmarking
- The cohorts lack a 'res' immunotherapy-response outcome column, making AUC benchmarking
  of signatures impossible
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: qs
  role: 'R package used to load the .qs cohort files: qread(''Sample File.qs'') and
    qread(''ICI cohort gene pairs.qs'')'
- name: abess
  role: Feature-selection step (abess_fit with support.size=0:100) that defines key
    genes before base learners and signature benchmarking
- name: iMLGAM / iMLAGA
  role: R package providing basic_learner, basic_learner_predictor, GA_train, and
    predict to compute the iMLGAM riskscore benchmarked against the 12 signatures
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Language in which the 12 published signatures are computed from the cohort
    RNA-seq expression matrices and AUC comparisons are performed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:a8859a5475323634eb94f1c3cfa7e151df3fd35889f59c53045298ea506e8368
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - abess
  - iMLAGA
  - iMLGAM / iMLAGA
  - qs
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.125
n_claims: 8
---

# gene-expression-based-signature-computation

## Summary

Compute published immunotherapy response signatures (CAF, IFNγ, chemokines, Davoli_IS, CEP, CYT, TLS, TIDE, NLPR3, Roh_IS, Ayers_expIS, IPS) directly from cohort RNA-seq expression matrices and benchmark their predictive performance (AUC for ICB response) against the iMLGAM riskscore in training, validation, and test ICB cohorts.

## When to use

Use when you have RNA-seq expression matrices from ICB-treated cohorts with a 'res' response column and need to compute/benchmark published immune signatures or compare a new score's AUC against them.

## When NOT to use

- The input is already a base-learner prediction matrix (output of basic_learner_predictor) rather than a raw RNA-seq expression matrix — signature computation from expression is unnecessary there
- The analysis goal is CRISPR-screen gene prioritization or CEP55 mechanism work, not signature computation or ICB-response benchmarking
- The cohorts lack a 'res' immunotherapy-response outcome column, making AUC benchmarking of signatures impossible

## Inputs

- 'Sample File.qs' (qs-serialized list of cohort expression/response objects: trainset_7, trainset_3, testset)
- 'ICI cohort gene pairs.qs' (qs-serialized ICI cohort gene-pair data with 'res' outcome column)
- GA-fitted model object ga_fit from GA_train(trainres)
- Cohort RNA-seq expression matrices (samples × genes)

## Outputs

- riskscore column appended to trainres, validatres, and testres via predict(ga_fit, ...)
- Values of 12 published immune signatures (CAF, IFNγ, chemokines, Davoli_IS, CEP, CYT, TLS, TIDE, NLPR3, Roh_IS, Ayers_expIS, IPS) per cohort
- signature_comparison_table.csv contrasting iMLGAM performance against each of the 12 signatures across training, validation, and test cohorts

## How to apply

Load the cohort objects with the qs package via qread() from 'Sample File.qs' and 'ICI cohort gene pairs.qs', assigning dat$trainset_7, dat$trainset_3, and dat$testset as training, validation, and test cohorts, and recode the first column as 'res'. Score each cohort with the fitted genetic-algorithm ensemble using predict(ga_fit, cohort) to append the iMLGAM 'riskscore' column. In R, compute each of the 12 published immune signatures (CAF, IFNγ, chemokines, Davoli_IS, CEP, CYT, TLS, TIDE, NLPR3, Roh_IS, Ayers_expIS, IPS) from the cohort RNA-seq expression matrices, keeping row identity aligned with the cohort samples (e.g. via intersect on rownames). Then evaluate each signature and the iMLGAM riskscore per cohort for immunotherapy-response prediction against the 'res' outcome, and assemble a comparison table (signature_comparison_table.csv) across all three cohorts.

## Related tools

- **qs** (R package used to load the .qs cohort files: qread('Sample File.qs') and qread('ICI cohort gene pairs.qs'))
- **abess** (Feature-selection step (abess_fit with support.size=0:100) that defines key genes before base learners and signature benchmarking)
- **iMLGAM / iMLAGA** (R package providing basic_learner, basic_learner_predictor, GA_train, and predict to compute the iMLGAM riskscore benchmarked against the 12 signatures) — https://github.com/Yelab1994/iMLGAM
- **R** (Language in which the 12 published signatures are computed from the cohort RNA-seq expression matrices and AUC comparisons are performed)

## Examples

```
trainres$riskscore <- predict(ga_fit, trainres); validatres$riskscore <- predict(ga_fit, validatres); testres$riskscore <- predict(ga_fit, testres)
```

## Evaluation signals

- Each of trainres, validatres, and testres gains a numeric 'riskscore' column produced by predict(ga_fit, ...)
- The comparison table shows the iMLGAM score consistently outperforming each of the 12 published signatures (CAF, IFNγ, chemokines, Davoli_IS, CEP, CYT, TLS, TIDE, NLPR3, Roh_IS, Ayers_expIS, IPS) across training, testing, and validation cohorts
- Lower iMLGAM scores correlate with enhanced therapeutic responses, so directionality of the riskscore-outcome association is negative
- Sample rows used in scoring align with the cohort 'res' column via intersect(rownames(dat$trainset_7), rownames(trainres)) style joins
- signature_comparison_table.csv exists and contains per-cohort performance (e.g. AUC) for iMLGAM plus all 12 signatures

## Limitations

- The 11 RNA-seq cohorts cover only six cancer types; the iMLGAM score's pan-cancer predictive efficacy requires further validation in prospective ICB trials
- No large-scale, multi-center pan-cancer cohort validation exists, limiting generalizability of the benchmarked performance
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses

## Evidence

- [intro] The field continues to lack a universally applicable predictive marker for accurate ICB response forecasting across various cancer types: "The field continues to lack a universally applicable predictive marker for accurate ICB response forecasting across various cancer types"
- [abstract] iMLGAM scores exhibit superior predictive performance across independent cohorts: "iMLGAM scores exhibit superior predictive performance across independent cohorts"
- [methods] qread of ICI cohort gene pairs: "dat <- qread("ICI cohort gene pairs.qs")"
- [readme] Obtaining the evaluation score of the training set: "trainres $riskscore <- predict(ga_fit ,trainres)"
- [readme] Feature selection with ABESS: "abess_fit <- abess(trainset[,-1], trainset$res,support.size=0:100)"
