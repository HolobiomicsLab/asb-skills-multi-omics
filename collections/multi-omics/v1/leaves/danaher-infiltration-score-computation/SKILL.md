---
name: danaher-infiltration-score-computation
description: Use when a TCGA bulk RNA-seq cohort must be stratified by iMLGAM 'riskscore'
  (from predict(ga_fit, ...)) and immune infiltration — Danaher scores, leukocyte/lymphocyte/TIL
  fractions, CD8+ T cells — contrasted between score groups.
when_to_use_negative:
- Input is single-cell or imaging/IHC-derived immune quantification rather than bulk
  TCGA transcriptomic data scored against a pre-trained iMLGAM GA model
- No pre-trained ga_fit model or training-cohort cutoff is available, so scores cannot
  be computed with the original formula
- Goal is training a new iMLGAM model (base learners, ABESS feature selection, GA_train)
  rather than scoring and phenotyping an existing cohort
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: iMLGAM
  role: R package providing GA_train / predict(ga_fit, ...) to compute 'riskscore'
    used to stratify the TCGA cohort into high/low groups
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Runtime environment for loading .qs data, predicting scores, and computing/contrasting
    immune metrics
- name: qs
  role: Used to read the sample .qs files (qread) that carry the outcome ('res') and
    feature matrices
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:1936d38bb8f9b599f92a8a7c81565f7d7c5a3dce362f465aec234050437e8f62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - iMLGAM
  - qs
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# danaher-infiltration-score-computation

## Summary

A TCGA-based immune-infiltration analysis skill: compute iMLGAM risk scores for a pan-cancer cohort with a pre-trained GA model, stratify samples into high/low 'riskscore' groups, and contrast per-sample immune microenvironment metrics — including Danaher gene-expression infiltration scores, leukocyte/lymphocyte/TIL fractions, and CD8+ T cell abundance — to test whether low-score tumors are more immune-infiltrated.

## When to use

Use when a TCGA bulk RNA-seq cohort must be stratified by iMLGAM 'riskscore' (from predict(ga_fit, ...)) and immune infiltration — Danaher scores, leukocyte/lymphocyte/TIL fractions, CD8+ T cells — contrasted between score groups.

## When NOT to use

- Input is single-cell or imaging/IHC-derived immune quantification rather than bulk TCGA transcriptomic data scored against a pre-trained iMLGAM GA model
- No pre-trained ga_fit model or training-cohort cutoff is available, so scores cannot be computed with the original formula
- Goal is training a new iMLGAM model (base learners, ABESS feature selection, GA_train) rather than scoring and phenotyping an existing cohort

## Inputs

- Sample .qs data file (e.g. 'Sample File.qs' / 'ICI cohort gene pairs.qs') with outcome column 'res' and feature columns
- Pre-trained GA model object (ga_fit) from GA_train on the iMLGAM training-cohort prediction matrix
- TCGA pan-cancer transcriptomic dataset (RNA-seq; per Table S2 list of pan-cancer transcriptomic datasets in TCGA)

## Outputs

- Per-sample iMLGAM 'riskscore' column for the TCGA cohort with aligned 'res' outcome
- Per-sample immune microenvironment metrics: leukocyte/lymphocyte/TIL fractions, CD8+ T cell abundance, Danaher gene-expression infiltration scores, fibroblast abundance
- Per-sample immune signature scores, cytolytic activity, and chemokine expression levels
- Single results table with one row per immune metric and columns for group comparison statistics (high vs low iMLGAM score)

## How to apply

Load the sample .qs data with the qs package in R and the pre-trained GA model (ga_fit from GA_train), then compute iMLGAM risk scores for the TCGA cohort via predict(ga_fit, ...) as in the iMLGAM package, keeping the outcome ('res') column aligned per sample. Split the cohort into high- and low-score groups using the 'riskscore' column and the training-cohort formula and cutoff. Per sample, estimate immune microenvironment quantities: leukocyte/lymphocyte/TIL fractions, CD8+ T cell abundance, Danaher gene-expression infiltration scores, and fibroblast abundance, then derive immune signature scores, cytolytic activity, and chemokine expression from the TCGA transcriptomic data. Contrast each immune metric between the high- and low-iMLGAM-score groups and assemble all group differences into a single results table. The rationale is that iMLGAM scores predict ICB outcomes, so the comparison validates whether low-score tumors carry the expected inflammatory, CD8+ T cell–enriched phenotype.

## Related tools

- **iMLGAM** (R package providing GA_train / predict(ga_fit, ...) to compute 'riskscore' used to stratify the TCGA cohort into high/low groups) — https://github.com/Yelab1994/iMLGAM
- **R** (Runtime environment for loading .qs data, predicting scores, and computing/contrasting immune metrics)
- **qs** (Used to read the sample .qs files (qread) that carry the outcome ('res') and feature matrices)

## Evaluation signals

- Recovered contrasts match the reported direction of effect: specimens with reduced iMLGAM scores show an inflammatory profile with enhanced CD8+ T lymphocyte infiltration and elevated immunogenicity
- Significantly larger fractions of leukocytes, lymphocytes, and TILs in the low iMLGAM score group, together with a higher proportion of CD8+ T cells
- Output file exists with one row per immune metric and columns for the group comparison statistics
- 'riskscore' values reproduce the training-cohort formula (predict(ga_fit, ...)) and the 'res' column stays aligned per sample after group splitting

## Limitations

- The 11 RNA-seq cohorts analyzed represent a limited range of six cancer types, with the iMLGAM score's pan-cancer predictive efficacy requiring further validation through prospective ICB trials
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses
- No large-scale, multi-center pan-cancer cohort validation currently exists, limiting the model's generalizability

## Evidence

- [discussion] low-score group immune phenotype: "specimens with reduced iMLGAM scores exhibited an inflammatory immunological profile, marked by enhanced CD8+T lymphocyte infiltration and elevated immunogenicity"
- [abstract] abstract-level finding on infiltration: "tumors with low iMLGAM scores display distinctive immune microenvironment characteristics, including increased immune cell infiltration and amplified antitumor immune responses"
- [discussion] TCGA dataset scope: "The comprehensive TCGA database was examined to characterize immunotherapeutic response patterns across malignancies."
- [readme] data loading from .qs file: "dat <- qread("Sample File.qs")"
- [readme] risk score computation step: "validatres $riskscore <- predict(ga_fit ,validatres)"
