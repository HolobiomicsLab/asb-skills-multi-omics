---
name: biomarker-performance-comparison-across-cohorts
description: Use when a new biomarker or scoring system must be validated against
  established published signatures across multiple independent cohorts to demonstrate
  reported predictive performance.
when_to_use_negative:
- When the input is already a comparison table of signature performance values rather
  than raw cohort expression and prediction matrices
- When only a single cohort is available and no independent validation/test cohorts
  exist to support cross-cohort benchmarking
- When the goal is biomarker discovery or feature selection (use ABESS/gene-pairing
  workflow) rather than benchmarking an existing score against published signatures
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: iMLGAM
  role: R package providing GA_train, predict, and the riskscore that is benchmarked
    against the 12 published immunotherapy signatures
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Language in which the cohorts are loaded, the 12 published signatures are
    computed from RNA-seq expression matrices, and AUCs are evaluated
- name: iMLAGA
  role: Genetic-algorithm learning package required alongside iMLGAM for fitting the
    model whose score is compared
  repo: https://github.com/Yelab1994/iMLAGA
- name: qs
  role: Used with qread to load 'Sample File.qs' and 'ICI cohort gene pairs.qs' cohort
    files
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:509745846d28540f419da6f270d7b30ffe7343dc0718125f40f4389dd67bec9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - iMLAGA
  - iMLGAM
  - qs
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# biomarker-performance-comparison-across-cohorts

## Summary

Benchmarks a novel predictive score (e.g., the iMLGAM riskscore produced by predict(ga_fit, cohort)) head-to-head against 12 published immunotherapy response signatures (CAF, IFNγ, chemokines, Davoli_IS, CEP, CYT, TLS, TIDE, NLPR3, Roh_IS, Ayers_expIS, IPS) across independent training, validation, and test ICB cohorts, using AUC against the 'res' response outcome.

## When to use

Use when a new biomarker or scoring system must be validated against established published signatures across multiple independent cohorts to demonstrate superior predictive performance.

## When NOT to use

- When the input is already a comparison table of signature performance values rather than raw cohort expression and prediction matrices
- When only a single cohort is available and no independent validation/test cohorts exist to support cross-cohort benchmarking
- When the goal is biomarker discovery or feature selection (use ABESS/gene-pairing workflow) rather than benchmarking an existing score against published signatures

## Inputs

- 'Sample File.qs' containing cohort expression data (trainset_7, trainset_3, testset) read via qread
- 'ICI cohort gene pairs.qs' containing prediction matrices with the 'res' outcome column read via qread
- ga_fit object produced by GA_train on the merged base-learner prediction matrix
- Cohort RNA-seq expression matrices from which the 12 published signatures are computed

## Outputs

- signature_comparison_table.csv contrasting iMLGAM performance against each of the 12 published signatures across training, validation, and test cohorts
- Per-cohort AUC values for iMLGAM riskscore and each published signature

## How to apply

Load cohort expression and prediction-matrix files with the qs package (qread) from 'Sample File.qs' and 'ICI cohort gene pairs.qs', assigning trainset_7 (training), trainset_3 (validation), and testset (test). Score each cohort with the fitted genetic-algorithm model via predict(ga_fit, cohort) to obtain the iMLGAM riskscore column. Compute the 12 published immune signatures (CAF, IFNγ, chemokines, Davoli_IS, CEP, CYT, TLS, TIDE, NLPR3, Roh_IS, Ayers_expIS, IPS) directly from each cohort's RNA-seq expression matrix in R. Evaluate AUC for immunotherapy response of each signature and the iMLGAM riskscore per cohort using the 'res' outcome column, then assemble a comparison table across all three cohorts. The rationale is that the field lacks a universally applicable ICB response marker, so superiority must be shown consistently in training, testing, and validation cohorts rather than a single dataset.

## Related tools

- **iMLGAM** (R package providing GA_train, predict, and the riskscore that is benchmarked against the 12 published immunotherapy signatures) — https://github.com/Yelab1994/iMLGAM
- **R** (Language in which the cohorts are loaded, the 12 published signatures are computed from RNA-seq expression matrices, and AUCs are evaluated)
- **iMLAGA** (Genetic-algorithm learning package required alongside iMLGAM for fitting the model whose score is compared) — https://github.com/Yelab1994/iMLAGA
- **qs** (Used with qread to load 'Sample File.qs' and 'ICI cohort gene pairs.qs' cohort files)

## Examples

```
library(qs); library(iMLGAM); dat <- qread("Sample File.qs"); trainres$riskscore <- predict(ga_fit, trainres)
```

## Evaluation signals

- The assembled signature_comparison_table.csv must show the iMLGAM riskscore consistently outperforming each of the 12 published signatures across all three cohorts (training, validation, test)
- Each cohort must have an iMLGAM riskscore column successfully populated via predict(ga_fit, cohort)
- AUC values must be computed against the 'res' outcome column present in each prediction matrix loaded from 'ICI cohort gene pairs.qs'
- All 12 signatures (CAF, IFNγ, chemokines, Davoli_IS, CEP, CYT, TLS, TIDE, NLPR3, Roh_IS, Ayers_expIS, IPS) must appear in the comparison table for every cohort
- Row names used for merging prediction matrices (via intersect) must align between 'Sample File.qs' and 'ICI cohort gene pairs.qs' cohorts

## Limitations

- The 11 RNA-seq cohorts cover only six cancer types; pan-cancer predictive efficacy of the iMLGAM score requires further validation in prospective ICB trials
- No large-scale, multi-center pan-cancer cohort validation exists, limiting the model's generalizability
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses

## Evidence

- [methods] Load the cohort expression and prediction-matrix files: "Download sample files via this [Baidu Netdisk page](https://pan.baidu.com/s/1hlTEJXgVYTQfrrMBXHe-RA?pwd=s8hr) and load the necessary R packages."
- [readme] Riskscore computed via predict on fitted GA model: "trainres $riskscore <- predict(ga_fit ,trainres)"
- [abstract] iMLGAM outperforms existing clinical biomarkers: "iMLGAM scores exhibit superior predictive performance across independent cohorts, with lower scores correlating significantly with enhanced therapeutic responses and outperforming existing clinical"
- [intro] Lack of universal ICB response marker motivates benchmarking: "The field continues to lack a universally applicable predictive marker for accurate ICB response forecasting across various cancer types"
- [discussion] Limited cancer-type coverage of validation cohorts: "the 11 RNA‐seq cohorts analyzed represent a limited range of six cancer types, with the iMLGAM score's pan‐cancer predictive efficacy requiring further validation through prospective ICB trials"
