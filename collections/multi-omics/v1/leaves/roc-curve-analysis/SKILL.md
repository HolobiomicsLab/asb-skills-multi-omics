---
name: roc-curve-analysis
description: Use when continuous biomarker scores (e.g. iMLGAM riskscore, PD-L1, TMB)
  and binary ICB response labels ('res' column) are available per cohort and you must
  compare predictive performance via ROC/AUC.
when_to_use_negative:
- The input is not a cohort with a binary ICB response label ('res') and continuous
  biomarker scores — e.g. survival-only data would need Cox-based evaluation instead
- 'The iMLGAM model has not yet been fitted (Steps 1–4 of the README: basic_learner,
  GA_train) so no riskscore exists to evaluate'
- The goal is biomarker discovery or feature selection (use the ABESS feature selection
  step with support.size=0:100 instead)
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: iMLGAM
  role: R package whose GA-fitted model (ga_fit) produces the riskscore that is one
    of the three ROC-evaluated predictors
  repo: https://github.com/Yelab1994/iMLGAM
- name: qs
  role: Loads the .qs cohort files (qread) containing outcome ('res') and gene-pair
    features for each ICI cohort
- name: R
  role: Language in which ROC curves, AUC values, comparison table, and per-cohort
    ROC plots are computed and saved
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:fb89d128102b6a60d9242efa450cb205c1920b1ae152606cff8d6846bc1dc2d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - iMLGAM
  - qs
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# roc-curve-analysis

## Summary

Benchmark the iMLGAM risk score against standard ICB biomarkers (PD-L1 expression and TMB) by computing ROC curves and AUC values for each predictor in the Mariathasan, Braun, OAK, and Riaz immunotherapy cohorts. Serves to quantitatively verify the claim that iMLGAM outperforms existing clinical biomarkers.

## When to use

Use when continuous biomarker scores (e.g. iMLGAM riskscore, PD-L1, TMB) and binary ICB response labels ('res' column) are available per cohort and you must compare predictive performance via ROC/AUC.

## When NOT to use

- The input is not a cohort with a binary ICB response label ('res') and continuous biomarker scores — e.g. survival-only data would need Cox-based evaluation instead
- The iMLGAM model has not yet been fitted (Steps 1–4 of the README: basic_learner, GA_train) so no riskscore exists to evaluate
- The goal is biomarker discovery or feature selection (use the ABESS feature selection step with support.size=0:100 instead)

## Inputs

- qs-format cohort files ('Sample File.qs', 'ICI cohort gene pairs.qs') for the Mariathasan, Braun, OAK, and Riaz ICI cohorts
- Binary immunotherapy response column ('res') per sample
- Continuous predictor values per sample: iMLGAM riskscore, PD-L1 expression, TMB

## Outputs

- roc_auc_comparison.csv table of per-cohort AUC values for iMLGAM, PD-L1, and TMB
- ROC curve figures for all three biomarkers per cohort

## How to apply

First, in R, load each immunotherapy cohort with the qs package (qread on 'Sample File.qs' and 'ICI cohort gene pairs.qs'), setting the 'res' column as the outcome per the iMLGAM Step 4 scoring script. Compute iMLGAM risk scores for each set by predicting from the genetic-algorithm-fitted model (predict(ga_fit, ...)), which yields the riskscore used as a continuous predictor. For each cohort, compute ROC curves and AUC values for three continuous predictors — the iMLGAM riskscore, PD-L1 expression, and TMB — against the binary immunotherapy response 'res'. Combine the per-cohort AUCs into a single comparison table (roc_auc_comparison.csv) and check that the iMLGAM AUC exceeds both PD-L1 and TMB AUCs in every cohort. Finally, plot the ROC curves for all three biomarkers per cohort and save the figures alongside the comparison table.

## Related tools

- **iMLGAM** (R package whose GA-fitted model (ga_fit) produces the riskscore that is one of the three ROC-evaluated predictors) — https://github.com/Yelab1994/iMLGAM
- **qs** (Loads the .qs cohort files (qread) containing outcome ('res') and gene-pair features for each ICI cohort)
- **R** (Language in which ROC curves, AUC values, comparison table, and per-cohort ROC plots are computed and saved)

## Examples

```
testres$riskscore <- predict(ga_fit, testres)
```

## Evaluation signals

- In every cohort (Mariathasan, Braun, OAK, Riaz), the iMLGAM AUC exceeds both the PD-L1 and TMB AUCs in roc_auc_comparison.csv
- Each cohort's riskscore vector comes from predict(ga_fit, ...) applied to the same samples that carry the 'res' outcome column (row names match after intersect())
- ROC plots show three curves (iMLGAM, PD-L1, TMB) per cohort, matching the rows of the comparison table
- roc_auc_comparison.csv is saved alongside the ROC figures with one AUC per biomarker per cohort
- AUC values fall in [0, 1], with higher iMLGAM scores expected to correspond to worse response (lower scores correlate with enhanced therapeutic responses)

## Limitations

- The 11 RNA-seq cohorts analyzed represent a limited range of six cancer types; the iMLGAM score's pan-cancer predictive efficacy requires further validation through prospective ICB trials
- No large-scale, multi-center pan-cancer cohort validation currently exists, limiting the model's generalizability
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses

## Evidence

- [other] The iMLGAM score consistently exceeded both PD-L1 and TMB in ROC curve analyses across the Mariathasan, Braun, OAK, and Riaz cohorts.: [evidence span withheld — anchor status: not_found]
- [abstract] iMLGAM scores exhibit superior predictive performance across independent cohorts: "iMLGAM scores exhibit superior predictive performance across independent cohorts, with lower scores correlating significantly with enhanced therapeutic responses and outperforming existing clinical"
- [readme] testres$riskscore<- predict(ga_fit ,testres): "#Obtaining the evaluation score of the test set
testres$riskscore<- predict(ga_fit ,testres)"
- [methods] trainset[!is.na(trainset$res),]: "#Feature Selection using ABESS Algorithm
trainset <- trainset[!is.na(trainset$res),]"
