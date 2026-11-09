---
name: drug-activity-score-computation
description: Use when a personalized cancer model (CanSeer-style) needs quantitative
  drug effect scores from tissue mean IC50 values plus a patient's normalized RNA-seq
  expression of drug-target genes with known oncogene/TSG roles.
when_to_use_negative:
- The patient lacks normalized RNA-seq expression or druggable target coverage in
  the biomolecular network model — the article notes the published model 'lacked coverage
  for certain mutations in some patients, resulting in inconclusive outcomes'
- The goal is prioritizing synergy-based drug combinations — the study 'remains limited
  in prioritizing synergy-based drug combinations'
- Expression data is raw/un-normalized RNA-seq — the scoring equations require normalized
  gene expressions as input
edam_topics:
- http://edamontology.org/topic_0202
- http://edamontology.org/topic_3308
tools:
- name: CanSeer
  role: Source methodology whose Step 4 scoring equations (DAS, NDAS, DS) are reconstructed
    and implemented; couples drug activity scores with normalized patient gene expressions
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:d5eb2cbf57cf2c3db4a0a15af2d5bb3c58f6cfd5720db05b3c044ec15507f995
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BIRL/CanSeer
  tools:
  - CanSeer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0202
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# drug-activity-score-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruction of CanSeer's Step 4 drug-scoring module, which computes drug activity scores (DAS), normalized drug activity scores (NDAS), and per-patient drug scores (DS) by combining tissue mean IC50 values with a patient's normalized gene expression and the oncogene/tumor-suppressor role of each drug-target gene. It enables quantitative, patient-specific quantification of therapy response for repositioned FDA-approved drugs.

## When to use

Use when a personalized cancer model (CanSeer-style) needs quantitative drug effect scores from tissue mean IC50 values plus a patient's normalized RNA-seq expression of drug-target genes with known oncogene/TSG roles.

## When NOT to use

- The patient lacks normalized RNA-seq expression or druggable target coverage in the biomolecular network model — the article notes the published model 'lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes'
- The goal is prioritizing synergy-based drug combinations — the study 'remains limited in prioritizing synergy-based drug combinations'
- Expression data is raw/un-normalized RNA-seq — the scoring equations require normalized gene expressions as input

## Inputs

- Tissue mean IC50 values per drug (computed after outlier removal)
- Patient's normalized gene expression table (RNA-seq, e.g. CSV loaded with pandas)
- Oncogene / tumor-suppressor role annotation for each drug-target gene
- List of curated candidate drugs (e.g. ipatasertib, afuresertib, osimertinib)

## Outputs

- Normalized drug activity scores (NDAS) per drug per patient (e.g. ndas_scores.csv)
- Patient-specific drug scores (DS) used to annotate the personalized network models

## How to apply

Load the tissue mean IC50 values for each candidate drug and the patient's normalized gene expression table with pandas, then load the oncogene/tumor-suppressor role annotation for each drug-target gene. Use average IC50 values computed only after outlier removal, since CanSeer uses these averages to derive the normalized drug activity score (NDAS). Implement the Step 4 equations by weighting each target gene's normalized expression by the drug's tissue mean IC50 and applying a role-based sign adjustment, so that overexpressed oncogenes and under-expressed tumor suppressors contribute opposing directions to the score. Compute NDAS per drug and combine with the drug activity score and patient-normalized expressions to obtain per-patient drug scores (DS), which are subsequently used to annotate the personalized models before re-analysis. Write the computed scores (e.g. ndas_scores.csv) and validate against published reference values: ipatasertib 0.0474, afuresertib 0.0715, osimertinib 0.1377 in the LUSC case study, within a tight numerical tolerance.

## Related tools

- **CanSeer** (Source methodology whose Step 4 scoring equations (DAS, NDAS, DS) are reconstructed and implemented; couples drug activity scores with normalized patient gene expressions) — github.com/BIRL/CanSeer

## Examples

```
import pandas as pd; ndas = ((expr * ic50) * role_sign).sum(axis=1); ndas.to_csv('ndas_scores.csv')
```

## Evaluation signals

- Computed NDAS values match the reported LUSC reference values — ipatasertib 0.0474, afuresertib 0.0715, osimertinib 0.1377 — within a tight numerical tolerance (output-matches-reference check)
- IC50 averages are computed only after outlier removal, as the article specifies using average IC50 post outlier removal for NDAS
- Role-based sign adjustment is verified: overexpressed oncogenes and under-expressed tumor suppressors contribute opposing directions to the score
- Output file ndas_scores.csv contains one score per queried drug with the expected drug labels
- Resulting DS values are consistent with their downstream use — annotating the personalized models before re-analysis

## Limitations

- Published network models may lack coverage for certain patient mutations, yielding inconclusive outcomes that propagate into scoring
- The study remains limited in prioritizing synergy-based drug combinations, so scores are for single drugs, not validated synergy
- Patient-specific drug cocktail sensitivity revealed in silico requires further evaluation with tumor-on-chip models; no wet-lab validation of predictions yet
- Framework may under- or overfit the patient's profile without panomics data expansion

## Evidence

- [other] NDAS from average IC50 after outlier removal: [evidence span withheld — anchor status: not_found]
- [other] Scoring equation structure: [evidence span withheld — anchor status: not_found]
- [other] Reference validation values: [evidence span withheld — anchor status: not_found]
