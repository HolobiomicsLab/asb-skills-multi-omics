---
name: group-level-pathway-score-contrast
description: Use when per-sample iMLGAM risk scores are available for a TCGA-style
  pan-cancer cohort with somatic mutation and copy-number data, and the goal is group-level
  contrasts of pathway scores or genomic alterations between score strata.
when_to_use_negative:
- Input lacks per-sample iMLGAM risk scores, so no low/high group stratification can
  be defined
- Input is raw immunotherapy response prediction modeling (base learner construction,
  ABESS feature selection, GA training) rather than post-hoc group-level genomic contrasts
- Only single-cancer-type data without the somatic mutation and copy-number layers
  needed for SBS signature exposure and 9p24.1 CNV detection
edam_operation: http://edamontology.org/operation_2945
edam_topics: []
tools:
- name: iMLGAM
  role: R package providing the risk-score computation whose output stratifies samples
    into low/high groups for the contrasts
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Analysis environment for signature quantification, pathway scoring, group
    testing, and CSV assembly
- name: abess
  role: Loaded as a prerequisite in the iMLGAM workflow (feature-selection step precedes
    score calculation)
- name: qs
  role: Used to load the .qs sample and ICI cohort gene-pair files that carry the
    training/validation/test sets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:9c89b2124052a2e249a72d703337ca8994e325071b886de840f977fa2183304c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - abess
  - iMLGAM
  - qs
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# group-level-pathway-score-contrast

## Summary

Stratify pan-cancer tumors into low- versus high-iMLGAM risk-score groups and contrast per-sample pathway activity scores (TP53/cell-cycle vs WNT/HIPPO/MYC) plus genomic features (SBS10b/SBS3/SBS6 signatures, 9p24.1 CD274/PDCD1LG2 amplification) between groups, testing the reported directional enrichments.

## When to use

Use when per-sample iMLGAM risk scores are available for a TCGA-style pan-cancer cohort with somatic mutation and copy-number data, and the goal is group-level contrasts of pathway scores or genomic alterations between score strata.

## When NOT to use

- Input lacks per-sample iMLGAM risk scores, so no low/high group stratification can be defined
- Input is raw immunotherapy response prediction modeling (base learner construction, ABESS feature selection, GA training) rather than post-hoc group-level genomic contrasts
- Only single-cancer-type data without the somatic mutation and copy-number layers needed for SBS signature exposure and 9p24.1 CNV detection

## Inputs

- TCGA pan-cancer somatic mutation catalogues (per-sample)
- TCGA copy-number segment data (for 9p24.1 focal amplification detection)
- Per-sample iMLGAM risk scores (from predict(ga_fit, ...))
- Sample-to-score mapping supporting low vs high group stratification

## Outputs

- mutation_signature_pathway_contrasts.csv (single results table of all group-level contrast statistics)
- Per-sample SBS10b/SBS3/SBS6 signature exposure values
- Per-sample TP53, cell-cycle, WNT, HIPPO, and MYC pathway scores
- 9p24.1 (CD274/PDCD1LG2) amplification calls per sample

## How to apply

Load TCGA pan-cancer somatic mutation and copy-number segment data together with per-sample iMLGAM risk scores into R and split tumors into low- versus high-score groups. Quantify per-sample exposure of the SBS10b, SBS3, and SBS6 mutational signatures from the somatic mutation catalogues and test whether they are elevated in the low-score group. Compute TP53 and cell-cycle pathway activity scores per sample (expected higher in the low-score group) and WNT, HIPPO, and MYC enrichment scores (expected higher in the high-score group). Detect focal copy-number amplification at chromosome 9p24.1 spanning CD274 (PD-L1) and PDCD1LG2 (PD-L2) and test its specificity to the low-score group. The rationale is that low iMLGAM scores mark an inflammatory, immunogenic TIME, so genomic determinants should segregate directionally by group. Assemble all group-level contrast statistics into a single results table (mutation_signature_pathway_contrasts.csv).

## Related tools

- **iMLGAM** (R package providing the risk-score computation whose output stratifies samples into low/high groups for the contrasts) — https://github.com/Yelab1994/iMLGAM
- **R** (Analysis environment for signature quantification, pathway scoring, group testing, and CSV assembly)
- **abess** (Loaded as a prerequisite in the iMLGAM workflow (feature-selection step precedes score calculation))
- **qs** (Used to load the .qs sample and ICI cohort gene-pair files that carry the training/validation/test sets)

## Examples

```
trainres$riskscore <- predict(ga_fit, trainres)
```

## Evaluation signals

- SBS10b, SBS3, and SBS6 signature frequencies are significantly higher in the low iMLGAM score group than in the high group
- TP53 and cell-cycle pathway activity scores are higher in the low-score group
- WNT, HIPPO, and MYC pathway enrichment scores are higher in the high-score group
- Focal 9p24.1 amplification spanning CD274 (PD-L1) and PDCD1LG2 (PD-L2) is detected specifically in the low-score group
- mutation_signature_pathway_contrasts.csv is written and every contrast reproduces the reported directional enrichment

## Limitations

- The 11 RNA-seq cohorts analyzed cover only six cancer types; pan-cancer predictive efficacy of the iMLGAM score requires validation in prospective ICB trials
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses
- No large-scale, multi-center pan-cancer cohort validation exists, limiting generalizability of score-group contrasts

## Evidence

- [discussion] low iMLGAM cohort immune profile: "specimens with reduced iMLGAM scores exhibited an inflammatory immunological profile, marked by enhanced CD8+T lymphocyte infiltration and elevated immunogenicity"
- [discussion] TCGA usage: "The comprehensive TCGA database was examined to characterize immunotherapeutic response patterns across malignancies."
- [readme] score calculation step: "testres$riskscore<- predict(ga_fit ,testres)"
- [readme] ABESS feature selection precedes scoring: "abess_fit <- abess(trainset[,-1], trainset$res,support.size=0:100)
key_gene <- extract(abess_fit, 5)[4][1]$support.vars"
- [methods] prerequisites: "Make sure you have R and the following packages installed: iMLAGA."
