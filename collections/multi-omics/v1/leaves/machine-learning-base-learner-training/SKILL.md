---
name: machine-learning-base-learner-training
description: 'Use when building the iMLGAM ensemble: training cohorts with an outcome
  column ''res'' and gene-pair feature columns need multiple base-learner models whose
  per-model predictions feed GA-based stacking.'
when_to_use_negative:
- Input is not a cohort split (70% trainset_7 / 30% trainset_3 / testset) with an
  outcome column named 'res' and gene-pair features — e.g. raw count matrices or single-sample
  data
- The goal is feature selection rather than learner training — ABESS (abess() with
  support.size=0:100, extract top 5 gene pairs) is a distinct upstream step
- Only a single machine-learning model is needed without ensemble stacking, since
  the method's premise is that ensemble learning outperforms individual approaches
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: iMLGAM
  role: R package providing basic_learner(), basic_learner_predictor(), and GA_train()
    for constructing base learners and optimizing the ensemble
  repo: https://github.com/Yelab1994/iMLGAM
- name: abess
  role: Adaptive best-subset selection that reduces the feature matrix to the 5 top-ranked
    gene pairs before base-learner construction
- name: qs
  role: Loads the .qs cohort files ('Sample File.qs', 'ICI cohort gene pairs.qs')
    into R data frames
- name: devtools
  role: Installs the iMLGAM package from GitHub prior to running the pipeline
  repo: https://github.com/Yelab1994/iMLGAM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:6e62418bbd132f65a4967026eda4b7ecd47d70703351664059f54c1c9233eba6
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# machine-learning-base-learner-training

## Summary

Trains an ensemble of diverse machine learning base learners on ABESS-selected gene-pair features from RNA-seq immunotherapy cohorts using the iMLGAM R package, then batches their predictions into matrices for genetic-algorithm-based ensemble optimization.

## When to use

Use when building the iMLGAM ensemble: training cohorts with an outcome column 'res' and gene-pair feature columns need multiple base-learner models whose per-model predictions feed GA-based stacking.

## When NOT to use

- Input is not a cohort split (70% trainset_7 / 30% trainset_3 / testset) with an outcome column named 'res' and gene-pair features — e.g. raw count matrices or single-sample data
- The goal is feature selection rather than learner training — ABESS (abess() with support.size=0:100, extract top 5 gene pairs) is a distinct upstream step
- Only a single machine-learning model is needed without ensemble stacking, since the method's premise is that ensemble learning outperforms individual approaches

## Inputs

- trainset / validationset / testset from 'Sample File.qs' (qread), each with first column 'res' (outcome) and gene-pair feature columns subset to the 5 ABESS-selected key genes
- basic_learner_list returned by basic_learner()
- 'ICI cohort gene pairs.qs' for merging response outcomes onto prediction matrices

## Outputs

- basic_learner_list: a list of trained base-learner models (indices such as m3, m5, m6, m16, m19, m23, m24, m29, m40, m46 survive GA selection)
- Per-cohort prediction matrices: trainres, validatres, testres with one prediction column per base-learner model, later augmented with 'res' and 'riskscore' columns

## How to apply

After ABESS feature selection (abess() with support.size=0:100, top 5 gene pairs extracted via extract(abess_fit, 5)), subset the trainset, validationset, and testset to 'res' plus the selected key_gene columns. Call basic_learner() on the training set with CVnumber=10, Cvrepeats=5, and ncore=8 so each candidate model is cross-validated with 10-fold CV repeated 5 times across 8 cores; the rationale is that ensemble learning outperforms individual ML approaches on transcriptomic data. Then batch-run basic_learner_predictor() on each of the three cohorts to produce per-model prediction matrices (trainres, validatres, testres). Merge the response outcomes from 'ICI cohort gene pairs.qs' onto these matrices by intersecting row names, which is required before GA_train() can select the optimal base-learner combination.

## Related tools

- **iMLGAM** (R package providing basic_learner(), basic_learner_predictor(), and GA_train() for constructing base learners and optimizing the ensemble) — https://github.com/Yelab1994/iMLGAM
- **abess** (Adaptive best-subset selection that reduces the feature matrix to the 5 top-ranked gene pairs before base-learner construction)
- **qs** (Loads the .qs cohort files ('Sample File.qs', 'ICI cohort gene pairs.qs') into R data frames)
- **devtools** (Installs the iMLGAM package from GitHub prior to running the pipeline) — https://github.com/Yelab1994/iMLGAM

## Examples

```
basic_learner_list <- basic_learner(trainingset=trainset, CVnumber=10, Cvrepeats=5, ncore=8)
```

## Evaluation signals

- basic_learner() must complete with CVnumber=10, Cvrepeats=5, ncore=8 without erroring on the training set
- basic_learner_predictor() must return three prediction matrices (trainres, validatres, testres) with one prediction column per base-learner model and row names matching cohort samples
- Outcome merging must succeed via intersect() of row names against 'ICI cohort gene pairs.qs', appending a 'res' column to each prediction matrix
- Downstream GA_train(trainres) must converge and predict(ga_fit, ...) must yield a numeric 'riskscore' for every sample in all three cohorts
- Under the paper's convention, lower iMLGAM scores should correspond to enhanced ICB therapeutic response

## Limitations

- Ensemble learning carries a risk of overfitting and must handle heterogeneity of sequencing platforms
- The 11 RNA-seq cohorts analyzed cover only six cancer types; pan-cancer predictive efficacy requires validation in prospective ICB trials
- No large-scale, multi-center pan-cancer cohort validation exists, limiting generalizability

## Evidence

- [methods] basic_learner_list <- basic_learner(trainingset=trainset , CVnumber=10, Cvrepeats=5, ncore=8): "basic_learner_list <- basic_learner(trainingset=trainset , CVnumber=10, Cvrepeats=5, ncore=8)"
- [readme] Obtaining the prediction results of each model in the training set: "trainres <- basic_learner_predictor(trainset, basic_learner_list )"
- [abstract] we developed an innovative R package called integrated Machine Learning and Genetic Algorithm-driven Multiomics analysis (iMLGAM): "we developed an innovative R package called integrated Machine Learning and Genetic Algorithm‐driven Multiomics analysis (iMLGAM)"
- [intro] ensemble learning methodologies have been demonstrated to provide enhanced predictive accuracy for outcomes when compared to individual machine learning approaches: "ensemble learning methodologies have been demonstrated to provide enhanced predictive accuracy for outcomes when compared to individual machine learning approaches"
- [other] the optimized iMLGAM score incorporated ten selected base-learner models (m3, m5, m6, m16, m19, m23, m24, m29, m40, m46): [evidence span withheld — anchor status: not_found]
