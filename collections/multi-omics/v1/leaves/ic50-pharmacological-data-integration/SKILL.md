---
name: ic50-pharmacological-data-integration
description: Use when you must rank candidate drugs for a personalized cancer model
  by converting drug IC50 sensitivity data plus patient-specific normalized expression
  into NDAS values.
when_to_use_negative:
- The published model lacks coverage for certain mutations in a patient, which the
  article reports results in inconclusive outcomes — scoring drugs against an incompletely
  annotated model is not meaningful.
- The goal is to prioritize drug-combination synergy; the article states the study
  remains limited in prioritizing synergy-based drug combinations.
- The input drug effect must be differentiated by mechanism for two independent drugs
  targeting the same gene — the prior in silico approaches CanSeer replaces could
  not do this, so verify your annotation carries drug-target-gene roles rather than
  assuming target identity alone suffices.
tools:
- name: CanSeer
  role: Translational methodology whose Step 4 scoring module computes DAS/NDAS from
    IC50 values, normalized gene expressions, and drug-target role annotations
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:7776f2bc528fe1d4f89771285765fd6776e42bbc76b0f9cd13f3a1e4f66d4d53
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BIRL/CanSeer
  repo_url: https://github.com/BIRL/CanSeer
  tools:
  - CanSeer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# ic50-pharmacological-data-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrates tissue mean IC50 pharmacological data with patient normalized gene expression and drug-target oncogene/tumor-suppressor annotations to compute drug activity scores (DAS) and normalized drug activity scores (NDAS) inside CanSeer's drug-scoring module.

## When to use

Use when you must rank candidate drugs for a personalized cancer model by converting drug IC50 sensitivity data plus patient-specific normalized expression into NDAS values.

## When NOT to use

- The published model lacks coverage for certain mutations in a patient, which the article reports results in inconclusive outcomes — scoring drugs against an incompletely annotated model is not meaningful.
- The goal is to prioritize drug-combination synergy; the article states the study remains limited in prioritizing synergy-based drug combinations.
- The input drug effect must be differentiated by mechanism for two independent drugs targeting the same gene — the prior in silico approaches CanSeer replaces could not do this, so verify your annotation carries drug-target-gene roles rather than assuming target identity alone suffices.

## Inputs

- Per-drug tissue mean IC50 values (post outlier-removal) table
- Patient normalized gene expression table (from patient RNA-seq data)
- Oncogene/tumor-suppressor role annotation for each drug-target gene
- Candidate drug list (e.g. ipatasertib, afuresertib, osimertinib)

## Outputs

- Normalized drug activity scores (NDAS) per drug per patient
- ndas_scores.csv containing the computed NDAS values
- Patient-specific drug scores (DS) used to annotate the personalized models

## How to apply

Load per-drug tissue mean IC50 values (computed after outlier removal) and the patient's normalized gene expression table with pandas, then attach the oncogene/tumor-suppressor role annotation for each drug-target gene. Implement the Step 4 scoring equations: weight each target gene's normalized expression by the tissue mean IC50 of the drug and apply a role-based sign adjustment, so that overexpressed oncogenes and under-expressed tumor suppressors contribute opposing directions to the score. These drug activity scores feed CanSeer's computation of patient-specific drug scores (DS), which are used to annotate the personalized models before re-analysis. Compute NDAS for each candidate drug and write the scores to ndas_scores.csv. In the LUSC case study the reference NDAS values are ipatasertib 0.0474, afuresertib 0.0715, and osimertinib 0.1377; validate your implementation against these within a tight numerical tolerance.

## Related tools

- **CanSeer** (Translational methodology whose Step 4 scoring module computes DAS/NDAS from IC50 values, normalized gene expressions, and drug-target role annotations) — github.com/BIRL/CanSeer

## Examples

```
import pandas as pd; ic50 = pd.read_csv('tissue_mean_ic50.csv'); expr = pd.read_csv('patient_expr_norm.csv'); ndas = compute_ndas(ic50, expr, roles); ndas.to_csv('ndas_scores.csv')
```

## Evaluation signals

- Computed NDAS matches reported reference values within a tight numerical tolerance: ipatasertib 0.0474, afuresertib 0.0715, osimertinib 0.1377 (output-matches-reference check).
- ndas_scores.csv is produced with one score per candidate drug for the patient.
- Tissue mean IC50 values were derived after outlier removal before weighting.
- Role-based sign adjustment is applied: overexpressed oncogenes and under-expressed tumor suppressors contribute opposing directions.
- Resulting drug activity scores are consistent with the DS values CanSeer uses to annotate and re-analyze the personalized models.

## Limitations

- Published model coverage gaps for certain patient mutations lead to inconclusive outcomes, limiting downstream drug scoring.
- The study remains limited in prioritizing synergy-based drug combinations.
- In silico predicted patient-specific drug cocktail sensitivity requires further evaluation with tumor-on-chip models; no wet lab validation of predicted efficacious drug combinations and repositioned drugs yet.

## Evidence

- [methods] Average IC50 values computed after outlier removal were utilized for computing normalized drug activity score (NDAS): [evidence span withheld — anchor status: not_found]
- [methods] the computed NDAS values must match the reported reference values (ipatasertib 0.0474, afuresertib 0.0715, osimertinib 0.1377): [evidence span withheld — anchor status: not_found]
- [methods] overexpressed oncogenes and under-expressed tumor suppressors contribute opposing directions: [evidence span withheld — anchor status: not_found]
