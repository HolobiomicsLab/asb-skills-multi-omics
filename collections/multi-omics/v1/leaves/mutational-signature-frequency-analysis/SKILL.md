---
name: mutational-signature-frequency-analysis
description: Use when somatic mutation catalogues and copy-number segment data are
  available per sample with a binary group assignment (e.g., low vs high iMLGAM risk
  score) and the goal is to test whether mutational signatures, pathway activities,
  or focal amplifications are directionally enriched in one group.
when_to_use_negative:
- Only bulk transcriptomic data are available without somatic mutation catalogues
  or copy-number segments — SBS exposures and 9p24.1 amplification cannot be computed
- The goal is de novo discovery of mutational signatures rather than testing frequencies
  of known signatures (SBS10b, SBS3, SBS6) between predefined groups
- No binary group stratification (e.g., no per-sample iMLGAM risk scores or other
  low/high grouping) exists to contrast against
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: iMLGAM
  role: R package used to compute the per-sample risk scores that define the low/high
    comparison groups for the signature-frequency contrasts
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Analysis environment in which mutation signature exposures, pathway scores,
    and CNV contrasts are computed and written to CSV
  repo: https://github.com/Yelab1994/iMLGAM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:4c25eb112002ea6465bde2ac175451fdb9dc9e332a86c468bf83d3b8479c9666
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - iMLGAM
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mutational-signature-frequency-analysis

## Summary

Contrasts the per-sample exposure of specific SBS mutational signatures (SBS10b, SBS3, SBS6) between predefined iMLGAM risk-score groups in the TCGA pan-cancer cohort, alongside pathway activity and focal CNV checks, to characterize the genomic determinants of the iMLGAM immunotherapy-response score.

## When to use

Use when somatic mutation catalogues and copy-number segment data are available per sample with a binary group assignment (e.g., low vs high iMLGAM risk score) and the goal is to test whether mutational signatures, pathway activities, or focal amplifications are directionally enriched in one group.

## When NOT to use

- Only bulk transcriptomic data are available without somatic mutation catalogues or copy-number segments — SBS exposures and 9p24.1 amplification cannot be computed
- The goal is de novo discovery of mutational signatures rather than testing frequencies of known signatures (SBS10b, SBS3, SBS6) between predefined groups
- No binary group stratification (e.g., no per-sample iMLGAM risk scores or other low/high grouping) exists to contrast against

## Inputs

- TCGA pan-cancer somatic mutation catalogue (per-sample SNV lists)
- TCGA copy-number segment data per sample
- Per-sample iMLGAM risk scores (riskscore from predict(ga_fit, ...))
- R environment with the iMLGAM package installed via devtools::install_github('Yelab1994/iMLGAM')

## Outputs

- mutation_signature_pathway_contrasts.csv — table of group-level contrast statistics for SBS signature frequencies, pathway scores, and 9p24.1 amplification
- Per-sample SBS10b, SBS3, SBS6 signature exposure estimates
- Per-sample TP53/cell-cycle and WNT/HIPPO/MYC pathway enrichment scores

## How to apply

First stratify TCGA pan-cancer tumors into low- versus high-iMLGAM-score groups using per-sample risk scores computed with the iMLGAM R package (GA-trained model via predict(ga_fit, ...)). Then quantify per-sample exposure of the SBS10b, SBS3, and SBS6 mutational signatures from the somatic mutation catalogues and test whether their frequencies are elevated in the low-score group. In parallel, compute TP53 and cell-cycle pathway activity scores (expected higher in the low-score group) and WNT, HIPPO, and MYC pathway enrichment scores (expected higher in the high-score group). From the copy-number segment data, detect focal amplification at chromosome 9p24.1 spanning CD274 (PD-L1) and PDCD1LG2 (PD-L2) and test its specificity to the low-score group. Assemble all group-level contrast statistics into a single table written as mutation_signature_pathway_contrasts.csv, accepting it only when every contrast reproduces the reported directional enrichment.

## Related tools

- **iMLGAM** (R package used to compute the per-sample risk scores that define the low/high comparison groups for the signature-frequency contrasts) — https://github.com/Yelab1994/iMLGAM
- **R** (Analysis environment in which mutation signature exposures, pathway scores, and CNV contrasts are computed and written to CSV) — https://github.com/Yelab1994/iMLGAM

## Examples

```
Rscript -e 'library(iMLGAM); ga_fit <- GA_train(trainres); testres$riskscore <- predict(ga_fit, testres)'
```

## Evaluation signals

- SBS10b, SBS3, and SBS6 signature frequencies are significantly higher in the low iMLGAM score group than in the high group
- Focal copy-number amplification at 9p24.1 spanning CD274/PDCD1LG2 is detected specifically in the low-score group
- TP53 and cell-cycle pathway activity scores are higher in the low-score group
- WNT, HIPPO, and MYC pathway enrichment scores are higher in the high-score group
- Output file mutation_signature_pathway_contrasts.csv exists and contains one row per contrast with all directional enrichments reproduced

## Limitations

- The 11 RNA-seq cohorts cover only six cancer types; pan-cancer validity of iMLGAM-score group contrasts requires further validation in prospective ICB trials
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses
- No large-scale, multi-center pan-cancer cohort validation, limiting the generalizability of the group contrasts

## Evidence

- [results] SBS signature finding: [evidence span withheld — anchor status: not_found]
- [full_text] 9p24.1 CNV detection step: [evidence span withheld — anchor status: not_found]
- [readme] risk score computation from README: "testres$riskscore<- predict(ga_fit ,testres)"
- [abstract] low-score immune phenotype: "tumors with low iMLGAM scores display distinctive immune microenvironment characteristics, including increased immune cell infiltration and amplified antitumor immune responses"
