---
name: risk-score-based-cohort-stratification
description: Use when a per-sample continuous risk or predictive score (e.g. iMLGAM
  riskscore) is available for a cohort such as TCGA and the goal is to split tumors
  into low/high groups and test for differential genomic determinants.
when_to_use_negative:
- No per-sample continuous risk score exists for the cohort and none can be computed
  — stratification would be arbitrary
- The cohort lacks somatic mutation, copy-number, or expression data needed to compute
  the group-level genomic contrasts
- The analysis goal is ICB response prediction itself rather than post-hoc stratification
  and group comparison of an existing score
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: iMLGAM
  role: R package whose GA-trained model produces the per-sample riskscore used to
    stratify tumors into low- and high-score groups
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Analysis environment for stratifying samples, computing SBS signature exposures
    and pathway scores, detecting 9p24.1 amplification, and writing the contrast table
- name: abess
  role: Feature selection (support.size=0:100) that determines key genes feeding the
    base learners behind the risk score
- name: qs
  role: Loads the .qs sample files (Sample File.qs, ICI cohort gene pairs.qs) holding
    the training/validation/test sets and outcomes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:74e19e3b4dad13e665213d1f1302f008fbbb8f17de0a53d71345bbc89845ae88
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - abess
  - iMLGAM
  - qs
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# risk-score-based-cohort-stratification

## Summary

Stratify tumor samples into low- versus high-risk groups using a computed per-sample risk score (here the iMLGAM score), then contrast genomic and immune features (mutational signatures, pathway activity, focal CNV) between groups in R.

## When to use

Use when a per-sample continuous risk or predictive score (e.g. iMLGAM riskscore) is available for a cohort such as TCGA and the goal is to split tumors into low/high groups and test for differential genomic determinants.

## When NOT to use

- No per-sample continuous risk score exists for the cohort and none can be computed — stratification would be arbitrary
- The cohort lacks somatic mutation, copy-number, or expression data needed to compute the group-level genomic contrasts
- The analysis goal is ICB response prediction itself rather than post-hoc stratification and group comparison of an existing score

## Inputs

- Per-sample iMLGAM risk scores (riskscore column produced by predict(ga_fit, ...))
- TCGA pan-cancer somatic mutation catalogues (per-sample SNV/indel data for signature exposure)
- TCGA copy-number segment data (for 9p24.1 focal amplification detection)
- Gene expression matrix (for TP53/cell-cycle and WNT/HIPPO/MYC pathway activity scoring)

## Outputs

- Group-level contrast statistics table (mutation_signature_pathway_contrasts.csv) covering SBS10b/SBS3/SBS6, TP53/cell-cycle, WNT/HIPPO/MYC, and 9p24.1 amplification contrasts
- Low- versus high-iMLGAM-score sample group assignments

## How to apply

Load per-sample risk scores (e.g. iMLGAM riskscore computed via GA_train and predict() in R) together with the cohort's somatic mutation catalogues and copy-number segment data, and split tumors into low- versus high-score groups. Quantify per-sample exposure of specific SBS mutational signatures (SBS10b, SBS3, SBS6) from the mutation catalogues and test whether their frequencies are elevated in the low-score group. Compute per-sample TP53 and cell-cycle pathway activity scores, expecting higher values in the low-score group, and WNT, HIPPO, and MYC enrichment scores, expecting enrichment in the high-score group. Detect focal copy-number amplification at chromosome 9p24.1 spanning CD274 (PD-L1) and PDCD1LG2 (PD-L2) and test its specificity to the low-score group. Assemble all group-level contrast statistics into a single results table (e.g. mutation_signature_pathway_contrasts.csv) and validate that each contrast reproduces the reported directional enrichment.

## Related tools

- **iMLGAM** (R package whose GA-trained model produces the per-sample riskscore used to stratify tumors into low- and high-score groups) — https://github.com/Yelab1994/iMLGAM
- **R** (Analysis environment for stratifying samples, computing SBS signature exposures and pathway scores, detecting 9p24.1 amplification, and writing the contrast table)
- **abess** (Feature selection (support.size=0:100) that determines key genes feeding the base learners behind the risk score)
- **qs** (Loads the .qs sample files (Sample File.qs, ICI cohort gene pairs.qs) holding the training/validation/test sets and outcomes)

## Examples

```
library(qs); library(abess); library(iMLGAM); dat <- qread("Sample File.qs"); ga_fit <- GA_train(trainres); riskscore <- predict(ga_fit, testres)
```

## Evaluation signals

- SBS10b, SBS3, and SBS6 signature frequencies are significantly higher in the low iMLGAM score group than the high group
- TP53 and cell-cycle pathway activity scores are higher in the low-score group
- WNT, HIPPO, and MYC pathway enrichment scores are higher in the high-score group
- 9p24.1 focal amplification spanning CD274/PDCD1LG2 is detected predominantly in the low-score group
- Every row of mutation_signature_pathway_contrasts.csv reproduces the reported directional enrichment for its contrast

## Limitations

- iMLGAM score's pan-cancer predictive efficacy requires validation in prospective ICB trials; the 11 RNA-seq cohorts cover only six cancer types
- No large-scale, multi-center pan-cancer cohort validation exists, limiting generalizability of the score and its stratification
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses across stratified groups

## Evidence

- [other] SBS10b, SBS3, SBS6 in low-score group: [evidence span withheld — anchor status: not_found]
- [abstract] low-score immune profile: "tumors with low iMLGAM scores display distinctive immune microenvironment characteristics, including increased immune cell infiltration and amplified antitumor immune"
- [readme] risk score computation: "testres$riskscore<- predict(ga_fit ,testres)"
- [methods] abess feature selection: "abess_fit <- abess(trainset[,-1], trainset$res,support.size=0:100)"
- [discussion] TCGA cohort: "The comprehensive TCGA database was examined to characterize immunotherapeutic response patterns across malignancies."
