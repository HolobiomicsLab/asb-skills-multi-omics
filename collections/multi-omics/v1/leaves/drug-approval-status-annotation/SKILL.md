---
name: drug-approval-status-annotation
description: Use when you have a list of predicted monotherapies or drug combinations
  from an in silico therapeutic prioritization (e.g. CanSeer drug-score output) and
  need to classify each entry by FDA approval status, literature support, and novelty.
when_to_use_negative:
- Prioritizing synergy-based drug combinations — the article states 'the study remains
  limited in prioritizing synergy-based drug combinations', so this annotation skill
  should not be treated as a synergy-ranking method
- Validating predicted efficacious drug combinations or repositioned drugs experimentally
  — wet-lab validation is explicitly flagged as still needed and out of scope here
- Input is patient-specific genetic alterations/RNA-seq data for building personalized
  network models — use CanSeer's model-construction workflow instead of approval-status
  annotation
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3379
tools:
- name: CanSeer
  role: Source of the predicted mono-targeted drugs, drug combinations, and FDA-approved
    repositioning candidates that are subsequently annotated and categorized by approval
    status
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:785f3f2233027d96a3c3e05613d00041fc937a629498b27960a7246e95b38bda
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# drug-approval-status-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates drug and drug-combination predictions (e.g. from CanSeer's LUSC case study, Supplementary Data 15) with FDA approval status and prior-literature support, then buckets them into FDA-approved mono-targeted drugs, FDA-approved combinations, literature-corroborated entries, repositioning candidates, and novel combinations.

## When to use

Use when you have a list of predicted monotherapies or drug combinations from an in silico therapeutic prioritization (e.g. CanSeer drug-score output) and need to classify each entry by FDA approval status, literature support, and novelty.

## When NOT to use

- Prioritizing synergy-based drug combinations — the article states 'the study remains limited in prioritizing synergy-based drug combinations', so this annotation skill should not be treated as a synergy-ranking method
- Validating predicted efficacious drug combinations or repositioned drugs experimentally — wet-lab validation is explicitly flagged as still needed and out of scope here
- Input is patient-specific genetic alterations/RNA-seq data for building personalized network models — use CanSeer's model-construction workflow instead of approval-status annotation

## Inputs

- Predicted single-drug list from CanSeer LUSC case study (Supplementary Data 15)
- Predicted drug-combination list from CanSeer LUSC case study (Supplementary Data 15)
- CanSeer FDA-approved drug repositioning output
- FDA approval status reference for oncology drugs/combinations

## Outputs

- drug_classification.csv — categorized table with per-entry FDA approval status and category labels
- Category counts: 4 FDA-approved mono-targeted drugs, 2 FDA-approved combinations, 26 literature-corroborated entries, 20 repositioning candidates, 97 novel combinations

## How to apply

Load the predicted single-drug and drug-combination lists (CanSeer LUSC case study, Supplementary Data 15, the source of the Fig. 6 categorization). Annotate each entry with its FDA approval status, flagging the four FDA-approved mono-targeted drugs (Afatinib, Osimertinib, Dabrafenib, Trametinib) and the two FDA-approved combinations (Osimertinib + Oxaliplatin, Dabrafenib + Trametinib). Cross-reference remaining entries against prior literature to label the 26 literature-corroborated drugs/combinations, and label the 20 repositioning candidates (FDA-approved drugs repositioned for LUSC) from the CanSeer repositioning output. Assign the 97 remaining combinations lacking prior support (e.g. Afuresertib + Palbociclib, Dinaciclib + Trametinib, Afatinib + Oxaliplatin, Ulixertinib + Olaparib) as novel combinations. Verify per-category counts equal the reported values (4, 2, 26, 20, 97) and write the categorized table to drug_classification.csv.

## Related tools

- **CanSeer** (Source of the predicted mono-targeted drugs, drug combinations, and FDA-approved repositioning candidates that are subsequently annotated and categorized by approval status) — github.com/BIRL/CanSeer

## Evaluation signals

- Per-category counts match reported values: 4 FDA-approved mono-targeted drugs, 2 FDA-approved combinations, 26 literature-corroborated, 20 repositioning candidates, 97 novel combinations
- The four FDA-approved mono-targeted drugs are exactly Afatinib, Osimertinib, Dabrafenib, and Trametinib
- The two FDA-approved combinations are exactly Osimertinib + Oxaliplatin and Dabrafenib + Trametinib
- Novel-combination bucket includes the exemplars Afuresertib + Palbociclib, Dinaciclib + Trametinib, Afatinib + Oxaliplatin, and Ulixertinib + Olaparib
- drug_classification.csv is written with every entry from Supplementary Data 15 assigned to exactly one category

## Limitations

- The study remains limited in prioritizing synergy-based drug combinations
- No wet-lab validation of predicted efficacious drug combinations and repositioned drugs yet
- Navigating lengthy regulatory processes for novel drug combinations can delay clinical deployment, rendering an impediment in translation of in silico predictions into practice

## Evidence
