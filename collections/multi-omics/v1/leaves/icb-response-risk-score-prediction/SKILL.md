---
name: icb-response-risk-score-prediction
description: Use when RNA-seq cohorts from ICB/immunotherapy-treated patients (with
  a 'res' response outcome column) need a predictive risk score and it must be compared
  head-to-head with published immune signatures such as TIDE, IFNγ, CYT, or IPS.
when_to_use_negative:
- Input data are not ICB/immunotherapy-treated cohort RNA-seq with a response outcome
  ('res') column — the score's calibration and interpretation (lower = better response)
  are specific to ICB cohorts
- The goal is pan-cancer prognostic evaluation for a patient outside the six cancer
  types represented in the 11 RNA-seq cohorts, since pan-cancer efficacy still requires
  prospective ICB trial validation
- Only single-sample clinical assays (IHC, flow cytometry) are available without bulk
  RNA-seq expression matrices — signatures cannot be computed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: iMLGAM
  role: R package providing basic_learner, basic_learner_predictor, GA_train, and
    predict to generate the riskscore for each cohort
  repo: https://github.com/Yelab1994/iMLGAM
- name: iMLAGA
  role: Genetic-algorithm learning package installed as prerequisite for running GA_train
  repo: https://github.com/Yelab1994/iMLAGA
- name: qs
  role: Loads the cohort 'Sample File.qs' and 'ICI cohort gene pairs.qs' via qread
- name: abess
  role: 'Feature selection: abess(trainset[,-1], trainset$res, support.size=0:100)
    with extract(abess_fit, 5) to pick support.vars'
- name: devtools
  role: Installs the package via devtools::install_github("Yelab1994/iMLGAM")
  repo: https://github.com/Yelab1994/iMLGAM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:381ca1691bdf8cc7d0aa349979ebb923434ace90f8726a7d0f15ea18bc2fdbed
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# icb-response-risk-score-prediction

## Summary

Computes the iMLGAM riskscore for ICB-treated cohorts (training trainset_7, validation trainset_3, test testset) and benchmarks its predictive performance (e.g., AUC for immunotherapy response) against 12 published immunotherapy signatures, demonstrating that iMLGAM outperforms existing biomarkers across independent cohorts.

## When to use

Use when RNA-seq cohorts from ICB/immunotherapy-treated patients (with a 'res' response outcome column) need a predictive risk score and it must be compared head-to-head with published immune signatures such as TIDE, IFNγ, CYT, or IPS.

## When NOT to use

- Input data are not ICB/immunotherapy-treated cohort RNA-seq with a response outcome ('res') column — the score's calibration and interpretation (lower = better response) are specific to ICB cohorts
- The goal is pan-cancer prognostic evaluation for a patient outside the six cancer types represented in the 11 RNA-seq cohorts, since pan-cancer efficacy still requires prospective ICB trial validation
- Only single-sample clinical assays (IHC, flow cytometry) are available without bulk RNA-seq expression matrices — signatures cannot be computed

## Inputs

- 'Sample File.qs' cohort expression/outcome file (qread), with trainset_7, trainset_3, testset and 'res' outcome column
- 'ICI cohort gene pairs.qs' prediction-matrix file (qread) for merging with base-learner predictions
- Fitted GA model object (ga_fit) from GA_train
- RNA-seq expression matrices from which the 12 published signatures are computed

## Outputs

- riskscore column appended to trainres, validatres, and testres via predict(ga_fit, cohort)
- Per-cohort predictive performance (e.g., AUC for immunotherapy response) for iMLGAM and each of the 12 signatures
- signature_comparison_table.csv contrasting iMLGAM vs. CAF, IFNγ, chemokines, Davoli_IS, CEP, CYT, TLS, TIDE, NLPR3, Roh_IS, Ayers_expIS, IPS across all three cohorts

## How to apply

Load the cohort expression and prediction-matrix files with the qs package (qread) from 'Sample File.qs' and 'ICI cohort gene pairs.qs', assigning dat$trainset_7, dat$trainset_3, and dat$testset as training, validation, and test cohorts. Build the model through the iMLGAM pipeline: ABESS feature selection (abess with support.size=0:100, extracting 5 support.vars), base learner construction (basic_learner with CVnumber=10, Cvrepeats=5), prediction matrices (basic_learner_predictor), and genetic-algorithm fitting (GA_train). Score each cohort with predict(ga_fit, cohort) to obtain the iMLGAM riskscore column, remembering that lower scores correlate with enhanced therapeutic response. Independently compute the 12 published immune signatures (CAF, IFNγ, chemokines, Davoli_IS, CEP, CYT, TLS, TIDE, NLPR3, Roh_IS, Ayers_expIS, IPS) from the same RNA-seq expression matrices in R, then evaluate each signature's AUC for immunotherapy response against the 'res' outcome column per cohort. Assemble the comparison table to confirm iMLGAM consistently outperforms all 12 signatures across training, testing, and validation cohorts.

## Related tools

- **iMLGAM** (R package providing basic_learner, basic_learner_predictor, GA_train, and predict to generate the riskscore for each cohort) — https://github.com/Yelab1994/iMLGAM
- **iMLAGA** (Genetic-algorithm learning package installed as prerequisite for running GA_train) — https://github.com/Yelab1994/iMLAGA
- **qs** (Loads the cohort 'Sample File.qs' and 'ICI cohort gene pairs.qs' via qread)
- **abess** (Feature selection: abess(trainset[,-1], trainset$res, support.size=0:100) with extract(abess_fit, 5) to pick support.vars)
- **devtools** (Installs the package via devtools::install_github("Yelab1994/iMLGAM")) — https://github.com/Yelab1994/iMLGAM

## Examples

```
trainres$riskscore <- predict(ga_fit, trainres)
```

## Evaluation signals

- The comparison table must show the iMLGAM riskscore consistently outperforming each of the 12 published signatures in predictive performance across training, testing, and validation cohorts
- Each cohort's scored object must contain a riskscore column produced by predict(ga_fit, cohort) on the merged prediction matrix
- Lower iMLGAM scores must correlate with enhanced therapeutic response ('res'), consistent with the reported direction of association
- ABESS feature selection must yield 5 key genes (extract(abess_fit, 5)) and all three cohorts must be subset to c('res', key_gene)
- Row-name intersections between 'ICI cohort gene pairs.qs' outcomes and each prediction matrix must be non-empty before cbind merging

## Limitations

- The 11 RNA-seq cohorts cover only six cancer types; the iMLGAM score's pan-cancer predictive efficacy requires further validation through prospective ICB trials
- No large-scale, multi-center pan-cancer cohort validation currently exists, limiting the model's generalizability
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses

## Evidence

- [results] When 12 published immunotherapy response signatures were evaluated alongside the iMLGAM score, the iMLGAM score consistently outperformed all of them in predictive performance across the training, testing, and validation cohorts.: [evidence span withheld — anchor status: not_found]
- [readme] Scoring each set using the formula constructed in Step 3.: "testres$riskscore<- predict(ga_fit ,testres)"
- [abstract] iMLGAM scores outperform existing clinical biomarkers across independent cohorts.: "iMLGAM scores exhibit superior predictive performance across independent cohorts, with lower scores correlating significantly with enhanced therapeutic responses"
- [readme] Cohort loading from qs files.: "dat <- qread("Sample File.qs")"
- [methods] Feature selection step.: "abess_fit <- abess(trainset[,-1], trainset$res,support.size=0:100)"
