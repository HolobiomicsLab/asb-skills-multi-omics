---
name: oncogene-tumor-suppressor-role-adjusted-scoring
description: Use when a patient-normalized expression table and per-drug target gene
  annotations (oncogene vs tumor suppressor) with tissue mean IC50 values must be
  combined into quantitative drug activity scores.
when_to_use_negative:
- When the goal is to compute treatment efficacy or cytotoxic effect from network
  model cell fate outcomes rather than activity-score-based drug scores
- When drug-target genes lack oncogene/tumor-suppressor role annotations, since the
  role-based sign adjustment is central to the formula
- When IC50 data have not undergone outlier removal, as the reported NDAS reference
  values are derived from post-outlier-removal averages
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0208
tools:
- name: CanSeer
  role: Pipeline whose Step 4 computes drug activity scores and patient-specific drug
    scores (DS) from drug activity scores and normalized gene expressions to annotate
    personalized models
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:398935422aeaaad267dc2dd582f1bf8f60570c4ba6e337272390ba15fb999b82
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0208
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# oncogene-tumor-suppressor-role-adjusted-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs CanSeer's Step 4 drug-scoring module that converts tissue mean IC50 values and patient-normalized gene expression into normalized drug activity scores (NDAS), applying a role-based sign adjustment for oncogene versus tumor-suppressor drug targets.

## When to use

Use when a patient-normalized expression table and per-drug target gene annotations (oncogene vs tumor suppressor) with tissue mean IC50 values must be combined into quantitative drug activity scores.

## When NOT to use

- When the goal is to compute treatment efficacy or cytotoxic effect from network model cell fate outcomes rather than activity-score-based drug scores
- When drug-target genes lack oncogene/tumor-suppressor role annotations, since the role-based sign adjustment is central to the formula
- When IC50 data have not undergone outlier removal, as the reported NDAS reference values are derived from post-outlier-removal averages

## Inputs

- Tissue mean IC50 values per drug (averaged after outlier removal)
- Patient's normalized gene expression table (RNA-seq derived)
- Oncogene/tumor-suppressor role annotation for each drug-target gene
- Drug-target gene mapping per candidate drug (e.g. ipatasertib, afuresertib, osimertinib)

## Outputs

- Normalized drug activity score (NDAS) per candidate drug per patient
- ndas_scores.csv with computed NDAS values

## How to apply

Load the tissue mean IC50 values for each drug and the patient's normalized gene expression table with pandas. Load the oncogene/tumor-suppressor role annotation for each drug-target gene. Implement the Step 4 equations by weighting each target gene's normalized expression by the tissue mean IC50 of the drug and applying a role-based sign adjustment, so that overexpressed oncogenes and under-expressed tumor suppressors contribute in opposing directions. Note that the IC50 values entering the score are averages computed after outlier removal. Compute NDAS for each candidate drug and write the scores to ndas_scores.csv; validate against reported reference values (ipatasertib 0.0474, afuresertib 0.0715, osimertinib 0.1377).

## Related tools

- **CanSeer** (Pipeline whose Step 4 computes drug activity scores and patient-specific drug scores (DS) from drug activity scores and normalized gene expressions to annotate personalized models) — github.com/BIRL/CanSeer

## Examples

```
python compute_ndas.py --tissue-ic50 tissue_mean_ic50.csv --roles drug_target_roles.csv --expr patient_norm_expr.csv --out ndas_scores.csv
```

## Evaluation signals

- Computed NDAS matches reported reference values within tight numerical tolerance: ipatasertib 0.0474, afuresertib 0.0715, osimertinib 0.1377 (output-matches-reference comparison)
- Sign adjustment is correct: overexpressed oncogenes and under-expressed tumor suppressors contribute opposing directions to the score
- IC50 inputs are averages computed after outlier removal, not raw IC50 values
- Output ndas_scores.csv exists with one NDAS entry per queried drug (ipatasertib, afuresertib, osimertinib)

## Limitations

- NDAS ranks candidate drugs but was not wet-lab validated; predicted efficacious drugs and combinations require experimental validation to enhance clinical acceptability
- The published model used for demonstration lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes
- Scores depend on the availability of tissue mean IC50 data for the drug's target genes

## Evidence

- [intro] CanSeer computes drug scores (DS) for patients by employing the drug activity score and normalized gene expressions of each individual cancer patient: [evidence span withheld — anchor status: not_found]
- [results] Average IC50 values computed after outlier removal were utilized for computing normalized drug activity score (NDAS): [evidence span withheld — anchor status: not_found]
