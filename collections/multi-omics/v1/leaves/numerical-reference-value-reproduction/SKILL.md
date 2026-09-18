---
name: numerical-reference-value-reproduction
description: Use when reimplementing a published scoring pipeline (CanSeer NDAS/drug
  scores) and reported reference values must be reproduced numerically as an output-matches-reference
  validation check.
when_to_use_negative:
- The goal is novel drug-combination discovery or therapy-response simulation from
  the personalized network models, rather than reproducing the Step 4 drug-score computation
  itself
- Patient IC50 or normalized expression inputs are not yet preprocessed (outlier removal
  and expression normalization are prerequisites for the tissue mean IC50s used here)
- The published model lacks coverage for the patient's target mutations, which the
  paper reports can yield inconclusive outcomes
edam_topics:
- http://edamontology.org/topic_0401
tools:
- name: CanSeer
  role: Published methodology whose Step 4 drug-scoring equations (DAS, NDAS, MEGID,
    DS) are reimplemented and validated against reported reference NDAS values
  repo: https://github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:a4ac886774d84b36380177804323e9f502d35e1fc2480b9863c3dfa0d0c05e9f
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
  edam_topics:
  - http://edamontology.org/topic_0401
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# numerical-reference-value-reproduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reimplements CanSeer's drug-scoring module (DAS/NDAS, MEGID, DS) from the article's Step 4 equations and validates the implementation by reproducing the reported reference NDAS values (ipatasertib 0.0474, afuresertib 0.0715, osimertinib 0.1377) from the LUSC case study within a tight numerical tolerance.

## When to use

Use when reimplementing a published scoring pipeline (CanSeer NDAS/drug scores) and reported reference values must be reproduced numerically as an output-matches-reference validation check.

## When NOT to use

- The goal is novel drug-combination discovery or therapy-response simulation from the personalized network models, rather than reproducing the Step 4 drug-score computation itself
- Patient IC50 or normalized expression inputs are not yet preprocessed (outlier removal and expression normalization are prerequisites for the tissue mean IC50s used here)
- The published model lacks coverage for the patient's target mutations, which the paper reports can yield inconclusive outcomes

## Inputs

- Tissue mean IC50 values per drug (computed after outlier removal)
- Patient's normalized gene expression table
- Oncogene/tumor-suppressor role annotation for each drug-target gene

## Outputs

- ndas_scores.csv containing computed NDAS values per drug (e.g. ipatasertib, afuresertib, osimertinib)
- Comparison of computed scores against reported reference values within tolerance

## How to apply

Load the per-drug tissue mean IC50 values and the patient's normalized gene expression table with pandas; tissue mean IC50s are computed after outlier removal. Load the oncogene/tumor-suppressor role annotation for each drug-target gene, then implement Step 4 of the CanSeer pipeline: weight each target gene's normalized expression by the tissue mean IC50 of the drug and apply a role-based sign adjustment so that overexpressed oncogenes and under-expressed tumor suppressors contribute opposing directions to the score. Compute NDAS for each candidate drug (here ipatasertib, afuresertib, osimertinib) and write the results to ndas_scores.csv. Judge correctness by an output-matches-reference comparison against the article's reported NDAS values (ipatasertib 0.0474, afuresertib 0.0715, osimertinib 0.1377) within a tight numerical tolerance.

## Related tools

- **CanSeer** (Published methodology whose Step 4 drug-scoring equations (DAS, NDAS, MEGID, DS) are reimplemented and validated against reported reference NDAS values) — https://github.com/BIRL/CanSeer

## Examples

```
import pandas as pd; ic50 = pd.read_csv('tissue_mean_ic50.csv'); expr = pd.read_csv('patient_expr_norm.csv'); ndas = compute_ndas(ic50, expr, roles); ndas.to_csv('ndas_scores.csv')
```

## Evaluation signals

- Computed NDAS values match the reference values ipatasertib 0.0474, afuresertib 0.0715, and osimertinib 0.1377 within a tight numerical tolerance
- Tissue mean IC50 inputs are averages computed after outlier removal, not raw IC50 medians
- Role-based sign adjustment correctly assigns opposing contribution directions to overexpressed oncogenes versus under-expressed tumor suppressors
- Output file ndas_scores.csv exists with one score per candidate drug and the expected schema

## Limitations

- The exact closed-form DAS/NDAS/MEGID/DS equations must be reconstructed from the Step 4 description; abstract-level evidence only states that drug scores use the drug activity score and normalized gene expressions
- The paper reports that the published model used for demonstration lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes
- The study remains limited in prioritizing synergy-based drug combinations, so reproduced single-drug scores do not extend directly to combination efficacy

## Evidence

- [intro] CanSeer computes "drug scores" (DS) for patients by employing the drug activity score and normalized gene expressions of each individual cancer patient: [evidence span withheld — anchor status: not_found]
