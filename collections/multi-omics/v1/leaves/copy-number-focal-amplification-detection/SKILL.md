---
name: copy-number-focal-amplification-detection
description: Use when per-sample TCGA copy-number segment data are available alongside
  iMLGAM risk scores and the goal is to identify focal amplifications (e.g. 9p24.1
  CD274/PDCD1LG2) and test their enrichment in a defined tumor subgroup.
when_to_use_negative:
- Cohorts that lack copy-number segment data (transcriptome-only datasets) — focal
  amplification cannot be detected from expression alone
- Samples without an iMLGAM risk score, since the low/high group contrast defining
  the enrichment test cannot be formed
- Scoring ICB response itself — use the iMLGAM scoring pipeline (ABESS feature selection,
  base learners, GA_train) instead; this skill only characterizes genomic determinants
  downstream of the score
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0622
tools:
- name: iMLGAM
  role: Provides the iMLGAM risk scores used to stratify TCGA tumors into low- and
    high-score groups against which 9p24.1 amplification frequency is contrasted
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Execution environment for loading TCGA copy-number segments, stratifying samples,
    and computing the group-level amplification contrast
  repo: https://github.com/Yelab1994/iMLGAM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:dd0279d4d505480e49871984e6334f58ab2e8c2f9524e6cd8d6b83f2d611a96c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - iMLGAM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# copy-number-focal-amplification-detection

## Summary

Detects focal somatic copy-number amplifications — in this cohort the 9p24.1 amplicon spanning CD274 (PD-L1) and PDCD1LG2 (PD-L2) — from TCGA copy-number segment data, and tests whether the amplification is specific to the low iMLGAM score group. This links the iMLGAM risk stratification to immune-checkpoint-locus amplification.

## When to use

Use when per-sample TCGA copy-number segment data are available alongside iMLGAM risk scores and the goal is to identify focal amplifications (e.g. 9p24.1 CD274/PDCD1LG2) and test their enrichment in a defined tumor subgroup.

## When NOT to use

- Cohorts that lack copy-number segment data (transcriptome-only datasets) — focal amplification cannot be detected from expression alone
- Samples without an iMLGAM risk score, since the low/high group contrast defining the enrichment test cannot be formed
- Scoring ICB response itself — use the iMLGAM scoring pipeline (ABESS feature selection, base learners, GA_train) instead; this skill only characterizes genomic determinants downstream of the score

## Inputs

- TCGA pan-cancer somatic copy-number segment data
- Per-sample iMLGAM risk scores for tumor stratification
- R environment with the iMLGAM package installed

## Outputs

- Per-sample focal amplification calls at 9p24.1 spanning CD274 and PDCD1LG2
- Group-level contrast statistics (low vs. high iMLGAM score) for the 9p24.1 amplification
- Combined results table written as mutation_signature_pathway_contrasts.csv

## How to apply

Load the TCGA pan-cancer copy-number segment data and per-sample iMLGAM risk scores into R, and stratify tumors into low- versus high-iMLGAM-score groups. Scan the segment data for focal amplification at chromosome 9p24.1 spanning CD274 (PD-L1) and PDCD1LG2 (PD-L2). Test whether the amplification frequency is significantly higher in the low-score group, which is expected because the low iMLGAM cohort shows amplified expression of PD-L1 and PD-1. Assemble this contrast together with the other group-level contrasts (SBS10b/SBS3/SBS6 signatures, TP53/cell-cycle, WNT/HIPPO/MYC) into a single results table written as mutation_signature_pathway_contrasts.csv. Accept the analysis only if the 9p24.1 amplification reproduces the reported directional enrichment in the low-score group.

## Related tools

- **iMLGAM** (Provides the iMLGAM risk scores used to stratify TCGA tumors into low- and high-score groups against which 9p24.1 amplification frequency is contrasted) — https://github.com/Yelab1994/iMLGAM
- **R** (Execution environment for loading TCGA copy-number segments, stratifying samples, and computing the group-level amplification contrast) — https://github.com/Yelab1994/iMLGAM

## Evaluation signals

- 9p24.1 amplification frequency is significantly higher in the low iMLGAM score group than the high group (directional enrichment reproduced)
- Amplified interval spans both CD274 (PD-L1) and PDCD1LG2 (PD-L2), consistent with the low-score cohort's elevated PD-L1/PD-1 checkpoint expression
- All group-level contrast statistics (SBS10b/SBS3/SBS6, TP53/cell-cycle, WNT/HIPPO/MYC, 9p24.1) are assembled into one table saved as mutation_signature_pathway_contrasts.csv
- Directional consistency across the full contrast set: low-score group enriched for SBS10b/SBS3/SBS6, TP53/cell-cycle activity, and 9p24.1 amplification; high-score group enriched for WNT/HIPPO/MYC

## Limitations

- Some TCGA datasets lack critical clinical annotations, hindering comprehensive biomarker analyses
- The pan-cancer validation is limited: the cohorts cover only six cancer types across 11 RNA-seq cohorts, so amplification-frequency contrasts may not generalize to all malignancies
- No large-scale, multi-center pan-cancer cohort validation currently exists, limiting generalizability of the group contrast

## Evidence

- [discussion] The comprehensive TCGA database was examined to characterize immunotherapeutic response patterns across malignancies.: "The comprehensive TCGA database was examined to characterize immunotherapeutic response patterns across malignancies."
- [discussion] Enhanced cytotoxic T-cell presence and checkpoint molecule expression in the low iMLGAM cohort: "demonstrated not only enhanced cytotoxic T‐cell presence but also amplified expression of immune checkpoint molecules, encompassing PD‐L1, PD‐1, and CTLA‐4"
- [readme] Study purpose from repository README grounding the multiomics pan-cancer context: "The purpose of this study is to construct a signature based on RNA sequencing data to predict the outcomes of immunotherapy in patients with pan-cancer."
