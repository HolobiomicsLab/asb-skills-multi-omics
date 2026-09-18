---
name: roc-auc-immunotherapy-response-evaluation
description: Use when an ICB/immunotherapy response prediction score (e.g. iMLGAM
  riskscore) has been computed and you must verify its discriminative performance
  holds within individual tumor types (RCC, urothelial carcinoma, melanoma, gastric
  cancer, GBM) rather than only in pooled cohorts.
when_to_use_negative:
- When the input is raw expression data that has not yet been scored — feature selection,
  base-learner construction and GA optimization must be run first to obtain riskscore
  values
- When pooled-cohort performance is sufficient and no per-tumor-type breakdown is
  required
- When the cohort lacks a 'res' response outcome annotation, since responder/non-responder
  stratification and ROC-AUC cannot be computed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: iMLGAM
  role: R package providing basic_learner, basic_learner_predictor and GA_train to
    build the base-learner ensemble and compute the iMLGAM riskscore evaluated per
    tumor type
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Runtime environment for the evaluation workflow (qs, abess, devtools, iMLGAM
    packages)
- name: iMLAGA
  role: Dependency package installed via GitHub that backs the iMLGAM genetic-algorithm
    learner
  repo: https://github.com/Yelab1994/iMLAGA
- name: qs
  role: Used with qread() to load the .qs sample expression files and ICI cohort outcome
    annotations
- name: abess
  role: Feature selection on the training set with support.size=0:100, extracting
    5 support variables (key_gene) before scoring
- name: devtools
  role: Used to install the iMLGAM package from GitHub (devtools::install_github)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:275908562eb119dc3cb544b7aa2dd8db4ac64fdd3c7c05edcda04f0ee17d3fae
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 9
---

# roc-auc-immunotherapy-response-evaluation

## Summary

A per-tumor-type evaluation workflow for the iMLGAM immunotherapy risk score: compute risk scores on training/validation/test ICI cohorts, stratify samples by cancer type, quantify ROC-AUC, and test responder vs non-responder score differences (Wilcoxon rank-sum) to confirm the score generalizes beyond pooled cohorts.

## When to use

Use when an ICB/immunotherapy response prediction score (e.g. iMLGAM riskscore) has been computed and you must verify its discriminative performance holds within individual tumor types (RCC, urothelial carcinoma, melanoma, gastric cancer, GBM) rather than only in pooled cohorts.

## When NOT to use

- When the input is raw expression data that has not yet been scored — feature selection, base-learner construction and GA optimization must be run first to obtain riskscore values
- When pooled-cohort performance is sufficient and no per-tumor-type breakdown is required
- When the cohort lacks a 'res' response outcome annotation, since responder/non-responder stratification and ROC-AUC cannot be computed

## Inputs

- Sample File.qs — sample expression file with trainset_7 (training), trainset_3 (validation) and testset partitions, first column 'res' as response outcome
- ICI cohort gene pairs.qs — ICI cohort outcome annotations with 'res' column for merging prediction matrices

## Outputs

- Per-set risk scores (trainres$riskscore, validatres$riskscore, testres$riskscore) from the GA-optimized iMLGAM model
- imlgam_score_evaluation_by_cancer_type.csv — per-tumor-type ROC-AUC, responder vs non-responder comparison, and significance

## How to apply

Load the sample expression files and the ICI cohort outcome file with the qs package (qread), partitioning samples into trainset_7, trainset_3 and testset splits and using the 'res' column as the response outcome. Perform feature selection on the training set with the abess algorithm (support.size=0:100) and extract 5 support variables as key_gene, then subset all three sets to these genes. Build the base-learner ensemble with basic_learner (CVnumber=10, Cvrepeats=5, ncore=8), generate prediction matrices for each set with basic_learner_predictor, merge them with the ICI cohort 'res' annotations by intersecting row names, and run GA_train on the training prediction matrix to obtain the genetic-algorithm-optimized base-learner combination. Compute riskscore for each set via predict(ga_fit, ...), then stratify the risk scores by tumor type, compute per-tumor-type ROC-AUC, and compare responder vs non-responder scores with a Wilcoxon rank-sum test. Write the per-type summary (AUC, responder/non-responder comparison, significance) to imlgam_score_evaluation_by_cancer_type.csv; the expected pattern is lower iMLGAM scores in responders with AUC > 0.6 in RCC, urothelial carcinoma, melanoma and gastric cancer, and no significant difference in GBM (limited sample size).

## Related tools

- **iMLGAM** (R package providing basic_learner, basic_learner_predictor and GA_train to build the base-learner ensemble and compute the iMLGAM riskscore evaluated per tumor type) — https://github.com/Yelab1994/iMLGAM
- **R** (Runtime environment for the evaluation workflow (qs, abess, devtools, iMLGAM packages))
- **iMLAGA** (Dependency package installed via GitHub that backs the iMLGAM genetic-algorithm learner) — https://github.com/Yelab1994/iMLAGA
- **qs** (Used with qread() to load the .qs sample expression files and ICI cohort outcome annotations)
- **abess** (Feature selection on the training set with support.size=0:100, extracting 5 support variables (key_gene) before scoring)
- **devtools** (Used to install the iMLGAM package from GitHub (devtools::install_github))

## Examples

```
trainres$riskscore <- predict(ga_fit, trainres)
```

## Evaluation signals

- ROC-AUC > 0.6 with significantly lower iMLGAM scores in responders (p < 0.05, Wilcoxon rank-sum) for RCC, urothelial carcinoma, melanoma, and gastric cancer
- No significant responder vs non-responder score difference for glioblastoma multiforme (GBM), attributed to limited sample size
- Prediction matrices successfully merged with ICI cohort 'res' annotations via row-name intersection (intersect of rownames) for all three sets
- riskscore column present for trainres, validatres and testres after predict(ga_fit, ...)
- Output CSV imlgam_score_evaluation_by_cancer_type.csv contains one row per tumor type with AUC, responder/non-responder comparison, and significance

## Limitations

- No significant responder/non-responder difference was found in GBM, likely due to limited sample size
- The 11 RNA-seq cohorts analyzed cover only six cancer types, and the iMLGAM score's pan-cancer predictive efficacy requires further validation in prospective ICB trials
- No large-scale, multi-center pan-cancer cohort validation currently exists, limiting generalizability

## Evidence

- [readme] Feature selection with abess and GA-optimized scoring: "abess_fit <- abess(trainset[,-1], trainset$res,support.size=0:100)"
- [readme] Base-learner ensemble construction parameters: "basic_learner_list <- basic_learner(trainingset=trainset , CVnumber=10, Cvrepeats=5, ncore=8)"
- [readme] Genetic-algorithm optimization of base-learner combination: "ga_fit <- GA_train(trainres)"
- [abstract] Lower iMLGAM scores correlate with enhanced therapeutic response: "iMLGAM scores exhibit superior predictive performance across independent cohorts, with lower scores correlating significantly with enhanced therapeutic responses"
- [discussion] Limited cohort range caveat: "the 11 RNA‐seq cohorts analyzed represent a limited range of six cancer types, with the iMLGAM score's pan‐cancer predictive efficacy requiring further validation through prospective ICB trials"
