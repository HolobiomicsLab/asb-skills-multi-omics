---
name: fda-approved-drug-repositioning-classification
description: Use when a list of computationally predicted cancer drugs or drug combinations
  (e.g., from CanSeer Supplementary Data 15) must be annotated with FDA approval status
  and prior-literature support to separate repositioning candidates from novel hypotheses.
when_to_use_negative:
- The predictions come from a tool other than CanSeer or lack the Supplementary Data
  15 drug/combination lists, since category labels and counts are anchored to that
  output
- The goal is synergy-based prioritization of combinations rather than regulatory/literature
  classification — the study itself 'remains limited in prioritizing synergy-based
  drug combinations'
edam_operation: http://edamontology.org/operation_3715
edam_topics:
- http://edamontology.org/topic_0208
- http://edamontology.org/topic_3376
tools:
- name: CanSeer
  role: Produces the personalized LUSC predictions (repositioned FDA-approved drugs,
    novel drug combinations) that are subsequently categorized by FDA approval status
    and literature support
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:c1a86c9bffe319c58f953f69d997afeba9eac7222e50e0cc2cd626d217e60903
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
  edam_operation: http://edamontology.org/operation_3715
  edam_topics:
  - http://edamontology.org/topic_0208
  - http://edamontology.org/topic_3376
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# fda-approved-drug-repositioning-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Categorizes CanSeer-predicted single drugs and drug combinations for LUSC by their regulatory and literature support: FDA-approved mono-targeted drugs, FDA-approved combinations, literature-corroborated entries, repositioning candidates, and novel combinations.

## When to use

Use when a list of computationally predicted cancer drugs or drug combinations (e.g., from CanSeer Supplementary Data 15) must be annotated with FDA approval status and prior-literature support to separate repositioning candidates from novel hypotheses.

## When NOT to use

- The predictions come from a tool other than CanSeer or lack the Supplementary Data 15 drug/combination lists, since category labels and counts are anchored to that output
- The goal is synergy-based prioritization of combinations rather than regulatory/literature classification — the study itself 'remains limited in prioritizing synergy-based drug combinations'

## Inputs

- CanSeer predicted single-drug list for LUSC (Supplementary Data 15)
- CanSeer predicted drug-combination list for LUSC (Supplementary Data 15)
- FDA approval records for non-small cell lung cancer drugs
- Prior literature corpus for cross-referencing drug/combination support

## Outputs

- drug_classification.csv with per-category labels (FDA-approved mono, FDA-approved combination, literature-corroborated, repositioning candidate, novel combination)
- Verified category counts (4 FDA-approved mono drugs, 2 FDA-approved combinations, 26 literature-corroborated, 20 repositioning candidates, 97 novel combinations)

## How to apply

Load the predicted single-drug and drug-combination lists from Supplementary Data 15, the source of the Fig. 6 categorization. Annotate each entry with its FDA approval status, flagging the four FDA-approved mono-targeted drugs (Afatinib, Osimertinib, Dabrafenib, Trametinib) and the two FDA-approved combinations (Osimertinib + Oxaliplatin and Dabrafenib + Trametinib). Cross-reference the remaining entries against prior literature to label the 26 literature-corroborated drugs/combinations, then label the 20 repositioning candidates (FDA-approved drugs repositioned for LUSC) per the CanSeer repositioning output. Label the 97 remaining drug combinations lacking prior support (e.g., Afuresertib + Palbociclib, Dinaciclib + Trametinib, Afatinib + Oxaliplatin, Ulixertinib + Olaparib) as novel combinations. Verify per-category counts equal 4, 2, 26, 20, and 97 and write the categorized table to drug_classification.csv.

## Related tools

- **CanSeer** (Produces the personalized LUSC predictions (repositioned FDA-approved drugs, novel drug combinations) that are subsequently categorized by FDA approval status and literature support) — github.com/BIRL/CanSeer

## Examples

```
python classify_drugs.py --supplementary_data15 pred_lusc.xlsx --fda nsclc_fda_approved.tsv --literature prior_support/ --out drug_classification.csv
```

## Evaluation signals

- Category counts exactly match the reported values: 4 FDA-approved mono-targeted drugs, 2 FDA-approved combinations, 26 literature-corroborated, 20 repositioning candidates, 97 novel combinations
- Afatinib, Osimertinib, Dabrafenib, and Trametinib appear in the FDA-approved mono-targeted category
- Osimertinib + Oxaliplatin and Dabrafenib + Trametinib appear in the FDA-approved combination category
- Named novel combinations (Afuresertib + Palbociclib, Dinaciclib + Trametinib, Afatinib + Oxaliplatin, Ulixertinib + Olaparib) are labeled as lacking prior support
- Output file drug_classification.csv exists and every entry carries exactly one category label

## Limitations

- The study remains limited in prioritizing synergy-based drug combinations, so category labels do not encode synergy
- No wet lab validation of predicted efficacious drug combinations and repositioned drugs has been performed yet
- In silico predicted patient-specific drug cocktail sensitivity requires further evaluation with tumor-on-chip models
- Lengthy regulatory processes for novel drug combinations can delay clinical deployment of in silico predictions

## Evidence
