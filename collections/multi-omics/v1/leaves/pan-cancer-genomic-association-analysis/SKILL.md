---
name: pan-cancer-genomic-association-analysis
description: Use when TCGA pan-cancer somatic mutation, copy-number segment, and per-sample
  iMLGAM risk-score data are available and the goal is to test whether genomic features
  differ by risk-score group.
when_to_use_negative:
- 'The analysis goal is immunotherapy response prediction itself (use the iMLGAM scoring
  pipeline: abess feature selection, basic_learner, GA_train) rather than post-hoc
  genomic characterization of score groups.'
- No copy-number segment data are available, so the 9p24.1 CD274/PDCD1LG2 amplification
  contrast cannot be evaluated.
- Samples are single-cohort/single-cancer without pan-cancer scope; the skill is grounded
  in the TCGA pan-cancer cohort examined for iMLGAM immunotherapeutic response patterns.
tools:
- name: iMLGAM
  role: R package used to compute the per-sample iMLGAM risk scores used to stratify
    tumors into low/high groups before the genomic association contrasts
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Environment in which TCGA mutation/CNV data are loaded, score groups are stratified,
    signature/pathway contrasts are computed, and the CSV results table is written
  repo: https://github.com/Yelab1994/iMLGAM
- name: abess
  role: Feature-selection step (abess_fit <- abess(..., support.size=0:100)) within
    the iMLGAM pipeline that yields the key genes underlying the risk scores used
    for grouping
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:be50470557a61f1e3e72eeeb9b878780c567f9cde71f20802accdd5371cb1804
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - abess
  - iMLGAM
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# pan-cancer-genomic-association-analysis

## Summary

A pan-cancer genomic association workflow that contrasts somatic mutational signatures (SBS10b, SBS3, SBS6), focal copy-number amplification (9p24.1 spanning CD274/PDCD1LG2), and oncogenic pathway activities (TP53/cell-cycle vs WNT/HIPPO/MYC) between low- and high-iMLGAM-score tumor groups in TCGA.

## When to use

Use when TCGA pan-cancer somatic mutation, copy-number segment, and per-sample iMLGAM risk-score data are available and the goal is to test whether genomic features differ by risk-score group.

## When NOT to use

- The analysis goal is immunotherapy response prediction itself (use the iMLGAM scoring pipeline: abess feature selection, basic_learner, GA_train) rather than post-hoc genomic characterization of score groups.
- No copy-number segment data are available, so the 9p24.1 CD274/PDCD1LG2 amplification contrast cannot be evaluated.
- Samples are single-cohort/single-cancer without pan-cancer scope; the skill is grounded in the TCGA pan-cancer cohort examined for iMLGAM immunotherapeutic response patterns.

## Inputs

- TCGA pan-cancer somatic mutation catalogues (per-sample SBS mutation data)
- TCGA copy-number segment data
- Per-sample iMLGAM risk scores (e.g., from trainres$riskscore <- predict(ga_fit, trainres))
- Sample File.qs / ICI cohort gene pairs.qs annotation files loaded via library(qs)

## Outputs

- mutation_signature_pathway_contrasts.csv: a single table of group-level contrast statistics for SBS10b/SBS3/SBS6 signature frequencies, TP53/cell-cycle and WNT/HIPPO/MYC pathway scores, and 9p24.1 amplification status by iMLGAM score group

## How to apply

Load TCGA pan-cancer somatic mutation catalogues, copy-number segment data, and per-sample iMLGAM risk scores (computed via the iMLGAM R package's four-step pipeline: ABESS feature selection, base-learner construction, GA_train optimization, and predict(ga_fit, ...)) into R, then stratify tumors into low- versus high-score groups. Quantify per-sample exposure of SBS10b, SBS3, and SBS6 mutational signatures and test whether their frequencies are elevated in the low-score group. Compute TP53 and cell-cycle pathway activity scores (expected higher in the low-score group) and WNT/HIPPO/MYC pathway enrichment scores (expected higher in the high-score group), contrasting the two groups. From copy-number data, detect focal amplification at chromosome 9p24.1 spanning CD274 (PD-L1) and PDCD1LG2 (PD-L2) and test its specificity to the low-score group. Assemble all group-level contrast statistics into a single results table (mutation_signature_pathway_contrasts.csv).

## Related tools

- **iMLGAM** (R package used to compute the per-sample iMLGAM risk scores used to stratify tumors into low/high groups before the genomic association contrasts) — https://github.com/Yelab1994/iMLGAM
- **R** (Environment in which TCGA mutation/CNV data are loaded, score groups are stratified, signature/pathway contrasts are computed, and the CSV results table is written) — https://github.com/Yelab1994/iMLGAM
- **abess** (Feature-selection step (abess_fit <- abess(..., support.size=0:100)) within the iMLGAM pipeline that yields the key genes underlying the risk scores used for grouping)

## Examples

```
library(iMLGAM); ga_fit <- GA_train(trainres); trainres$riskscore <- predict(ga_fit, trainres)
```

## Evaluation signals

- SBS10b, SBS3, and SBS6 mutational signature frequencies are significantly higher in the low iMLGAM score group than in the high group.
- TP53 and cell-cycle pathway activity scores are higher in the low-score group, while WNT, HIPPO, and MYC enrichment scores are higher in the high-score group.
- Focal copy-number amplification at 9p24.1 spanning CD274 (PD-L1) and PDCD1LG2 (PD-L2) is detected and specific to the low-score group.
- All contrast statistics are assembled into a single results table and written as mutation_signature_pathway_contrasts.csv.
- Risk scores reproduce the iMLGAM scoring step: trainres$riskscore <- predict(ga_fit, trainres), applied consistently to training, validation, and test sets.

## Limitations

- Some TCGA datasets lack critical clinical annotations, hindering comprehensive biomarker analyses.
- The iMLGAM score's pan-cancer predictive efficacy is based on 11 RNA-seq cohorts covering only six cancer types and requires further validation in prospective ICB trials.
- No large-scale, multi-center pan-cancer cohort validation exists, limiting the model's generalizability.

## Evidence

- [discussion] The comprehensive TCGA database was examined to characterize immunotherapeutic response patterns across malignancies.: "The comprehensive TCGA database was examined to characterize immunotherapeutic response patterns across malignancies."
- [discussion] specimens with reduced iMLGAM scores exhibited an inflammatory immunological profile, marked by enhanced CD8+T lymphocyte infiltration and elevated immunogenicity: "specimens with reduced iMLGAM scores exhibited an inflammatory immunological profile, marked by enhanced CD8+T lymphocyte infiltration and elevated immunogenicity"
- [readme] Make sure you have R and the following packages installed: iMLAGA.: "Make sure you have R and the following packages installed: iMLAGA."
- [readme] abess(trainset[,-1], trainset$res,support.size=0:100): "abess_fit <- abess(trainset[,-1], trainset$res,support.size=0:100)"
- [readme] ## Step 4: Calculating Risk Scores for Each set: "## Step 4: Calculating Risk Scores for Each set"
