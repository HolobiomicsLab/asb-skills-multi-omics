---
name: immune-signature-scoring
description: Use when ICB/immunotherapy cohort RNA-seq data with a binary 'res' response
  column are available and you must score patients with the iMLGAM riskscore and/or
  published immune signatures (TIDE, CYT, IFNγ, TLS, IPS, expIS, etc.) and compare
  their predictive AUC.
when_to_use_negative:
- Your goal is only to compute a single published signature (e.g. TIDE) and no iMLGAM
  riskscore comparison or ga_fit model is involved
- The cohort data lack the 'res' immunotherapy response annotation, which the article
  notes hinders comprehensive biomarker analyses
- The prediction matrices have not been built (basic_learner_predictor outputs and
  GA_train merge with 'ICI cohort gene pairs.qs' not yet done), so predict(ga_fit,
  cohort) cannot be applied
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: iMLGAM
  role: R package providing the scoring system; predict(ga_fit, cohort) yields the
    iMLGAM riskscore compared against published signatures
  repo: https://github.com/Yelab1994/iMLGAM
- name: iMLAGA
  role: Companion package to install alongside iMLGAM; supplies GA_train and the genetic-algorithm
    machinery used to fit ga_fit before scoring
  repo: https://github.com/Yelab1994/iMLAGA
- name: qs
  role: R package used to load the .qs cohort files ('Sample File.qs', 'ICI cohort
    gene pairs.qs') via qread before scoring
- name: abess
  role: Feature selection of key gene pairs (extract(abess_fit, 5)) that constrain
    which features enter base learners and downstream signature scoring
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:c0fc4bb6300e9c1aefd55eee8c09d64eec20dca8844e2444fa42783d5f558260
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - abess
  - iMLAGA
  - iMLGAM
  - qs
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# immune-signature-scoring

## Summary

Benchmark the iMLGAM ensemble riskscore (built from ABESS-selected gene pairs and a genetic-algorithm-optimized combination of base learners) against 12 published immunotherapy response signatures (CAF, IFNγ, chemokines, Davoli_IS, CEP, CYT, TLS, TIDE, NLPR3, Roh_IS, Ayers_expIS, IPS) computed from cohort RNA-seq expression matrices, evaluating AUC for ICB response across training, validation, and test cohorts.

## When to use

Use when ICB/immunotherapy cohort RNA-seq data with a binary 'res' response column are available and you must score patients with the iMLGAM riskscore and/or published immune signatures (TIDE, CYT, IFNγ, TLS, IPS, expIS, etc.) and compare their predictive AUC.

## When NOT to use

- Your goal is only to compute a single published signature (e.g. TIDE) and no iMLGAM riskscore comparison or ga_fit model is involved
- The cohort data lack the 'res' immunotherapy response annotation, which the article notes hinders comprehensive biomarker analyses
- The prediction matrices have not been built (basic_learner_predictor outputs and GA_train merge with 'ICI cohort gene pairs.qs' not yet done), so predict(ga_fit, cohort) cannot be applied

## Inputs

- 'Sample File.qs' cohort file read with qs::qread(), containing trainset_7, trainset_3, and testset expression/outcome tables with a 'res' response column
- 'ICI cohort gene pairs.qs' file with ICI cohort outcome annotations merged into prediction matrices by intersecting row names
- Fitted GA model object ga_fit returned by GA_train(trainres)
- Cohort RNA-seq expression matrices from which the 12 published immune signatures are computed

## Outputs

- riskscore column appended to trainres, validatres, and testres prediction matrices
- Per-cohort AUC/ROC metrics for the iMLGAM riskscore and each of the 12 published signatures against the 'res' outcome
- signature_comparison_table.csv contrasting iMLGAM performance against each published signature across all three cohorts

## How to apply

Load the cohort data with the qs package via qread('Sample File.qs') and qread('ICI cohort gene pairs.qs'), assigning dat$trainset_7, dat$trainset_3, and dat$testset as training, validation, and test cohorts; the first column after removing columns 2-4 is the 'res' outcome. Score each cohort with the fitted genetic-algorithm model using predict(ga_fit, cohort) to append a riskscore column, per Step 4 of the README. Independently compute each of the 12 published signatures (CAF, IFNγ, chemokines, Davoli_IS, CEP, CYT, TLS, TIDE, NLPR3, Roh_IS, Ayers_expIS, IPS) from the same cohort RNA-seq expression matrices in R. Evaluate each signature and the iMLGAM riskscore for ICB response prediction (e.g. AUC against the 'res' column) in all three cohorts, then assemble a comparison table (signature_comparison_table.csv). Validation requires the iMLGAM riskscore to consistently outperform every published signature across training, validation, and test cohorts, and lower iMLGAM scores should correspond to enhanced therapeutic response.

## Related tools

- **iMLGAM** (R package providing the scoring system; predict(ga_fit, cohort) yields the iMLGAM riskscore compared against published signatures) — https://github.com/Yelab1994/iMLGAM
- **iMLAGA** (Companion package to install alongside iMLGAM; supplies GA_train and the genetic-algorithm machinery used to fit ga_fit before scoring) — https://github.com/Yelab1994/iMLAGA
- **qs** (R package used to load the .qs cohort files ('Sample File.qs', 'ICI cohort gene pairs.qs') via qread before scoring)
- **abess** (Feature selection of key gene pairs (extract(abess_fit, 5)) that constrain which features enter base learners and downstream signature scoring)

## Examples

```
library(qs); library(iMLGAM); dat <- qread("ICI cohort gene pairs.qs"); trainres$riskscore <- predict(ga_fit, trainres)
```

## Evaluation signals

- Each cohort table carries a riskscore column with one value per sample after predict(ga_fit, cohort) is run on trainres, validatres, and testres
- signature_comparison_table.csv contains one row per method (iMLGAM plus 12 signatures: CAF, IFNγ, chemokines, Davoli_IS, CEP, CYT, TLS, TIDE, NLPR3, Roh_IS, Ayers_expIS, IPS) and columns for training, validation, and test cohorts
- AUC of the iMLGAM riskscore exceeds that of every one of the 12 published signatures in each of the three cohorts, consistent with the reported finding
- Lower iMLGAM scores align with responders/enhanced therapeutic response (scores are risk-oriented, not response-oriented)
- Sample sets scored per cohort match the row-name intersections used when merging 'res' from 'ICI cohort gene pairs.qs' into the prediction matrices

## Limitations

- The 11 RNA-seq cohorts cover only six cancer types, so pan-cancer predictive efficacy of the iMLGAM score requires validation in prospective ICB trials
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses
- No large-scale, multi-center pan-cancer cohort validation exists, limiting generalizability of the benchmarked score

## Evidence

- [abstract] iMLGAM scores exhibit superior predictive performance across independent cohorts, with lower scores correlating significantly with enhanced therapeutic responses: "iMLGAM scores exhibit superior predictive performance across independent cohorts, with lower scores correlating significantly with enhanced therapeutic responses"
- [methods] ## Step 4: Calculating Risk Scores for Each set: "## Step 4: Calculating Risk Scores for Each set"
- [readme] trainres $riskscore <- predict(ga_fit ,trainres): "trainres $riskscore <- predict(ga_fit ,trainres)"
- [discussion] the 11 RNA‐seq cohorts analyzed represent a limited range of six cancer types, with the iMLGAM score's pan‐cancer predictive efficacy requiring further validation through prospective ICB trials: "the 11 RNA‐seq cohorts analyzed represent a limited range of six cancer types, with the iMLGAM score's pan‐cancer predictive efficacy requiring further validation"
