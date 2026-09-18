---
name: literature-corroboration-curation
description: Use when a computational method (e.g. CanSeer drug scores) has produced
  predicted mono-targeted drugs or drug combinations that must be triaged into FDA-approved,
  literature-corroborated, repositioned, or novel categories.
when_to_use_negative:
- When the goal is synergy quantification of drug pairs rather than approval/literature
  corroboration triage — the article states the study 'remains limited in prioritizing
  synergy-based drug combinations'.
- When input is raw patient genomic/RNA-seq data rather than an existing CanSeer drug-prediction
  list; the skill annotates already-predicted drugs, it does not generate predictions.
- When a drug/combination already has a definitive external regulatory annotation;
  re-deriving the FDA flag adds no value.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: CanSeer
  role: Source of the predicted mono-targeted drugs, drug combinations, and repositioned
    FDA-approved drugs that are subsequently categorized by FDA status and literature
    corroboration
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:ce8ffc25cfa45fb72e1c3e52043643df5e08114ebb3c2cb9198c2a32213e73bb
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# literature-corroboration-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies CanSeer-predicted LUSC drugs and drug combinations by crossing each prediction against FDA approval status and prior literature, yielding the categorization reported for the Fig. 6 case study (FDA-approved mono drugs, FDA-approved combinations, literature-corroborated, repositioning candidates, novel combinations).

## When to use

Use when a computational method (e.g. CanSeer drug scores) has produced predicted mono-targeted drugs or drug combinations that must be triaged into FDA-approved, literature-corroborated, repositioned, or novel categories.

## When NOT to use

- When the goal is synergy quantification of drug pairs rather than approval/literature corroboration triage — the article states the study 'remains limited in prioritizing synergy-based drug combinations'.
- When input is raw patient genomic/RNA-seq data rather than an existing CanSeer drug-prediction list; the skill annotates already-predicted drugs, it does not generate predictions.
- When a drug/combination already has a definitive external regulatory annotation; re-deriving the FDA flag adds no value.

## Inputs

- Predicted single-drug list for LUSC from Supplementary Data 15
- Predicted drug-combination list for LUSC from Supplementary Data 15
- CanSeer repositioning output (FDA-approved drugs repositioned for LUSC)
- FDA approval records for NSCLC/non-small cell lung cancer indications
- Prior literature (publications supporting 26 drugs/combinations)

## Outputs

- drug_classification.csv — per-entry table with FDA-approval flags and category labels (FDA-approved mono drug, FDA-approved combination, literature-corroborated, repositioning candidate, novel combination)
- Per-category counts (4 FDA-approved mono drugs, 2 FDA-approved combinations, 26 literature-corroborated, 20 repositioning candidates, 97 novel combinations)

## How to apply

Load the predicted single-drug and drug-combination lists from Supplementary Data 15, which is the source of the Fig. 6 categorization. Annotate each entry with its FDA approval status, first flagging the FDA-approved mono-targeted drugs (Afatinib, Osimertinib, Dabrafenib, Trametinib) and the two FDA-approved combinations (Osimertinib + Oxaliplatin and Dabrafenib + Trametinib). Cross-reference the remaining entries against prior literature to label the 26 literature-corroborated drugs/combinations, then label the 20 repositioning candidates (FDA-approved drugs repositioned for LUSC) per the CanSeer repositioning output. Any combinations left without prior support, such as Afuresertib + Palbociclib, Dinaciclib + Trametinib, Afatinib + Oxaliplatin, or Ulixertinib + Olaparib, are labeled as novel combinations. Verify per-category counts equal the reported values (4, 2, 26, 20, 97) before writing the categorized table to drug_classification.csv.

## Related tools

- **CanSeer** (Source of the predicted mono-targeted drugs, drug combinations, and repositioned FDA-approved drugs that are subsequently categorized by FDA status and literature corroboration) — github.com/BIRL/CanSeer

## Examples

```
python classify_drugs.py --supplementary-data15 predictions.xlsx --approved-mono Afatinib,Osimertinib,Dabrafenib,Trametinib --approved-combos "Osimertinib+Oxaliplatin","Dabrafenib+Trametinib" --output drug_classification.csv
```

## Evaluation signals

- Per-category counts must equal the reported values: 4 FDA-approved mono-targeted drugs, 2 FDA-approved combinations, 26 literature-corroborated drugs/combinations, 20 repositioning candidates, and 97 novel combinations.
- The four mono-targeted FDA-approved drugs in the output must be exactly Afatinib, Osimertinib, Dabrafenib, and Trametinib.
- The two FDA-approved combinations in the output must be exactly Osimertinib + Oxaliplatin and Dabrafenib + Trametinib.
- Every entry in Supplementary Data 15 must receive exactly one category label (no duplicates, no unclassified rows) before writing drug_classification.csv.
- Novel combinations must include the exemplars named in the paper: Afuresertib + Palbociclib, Dinaciclib + Trametinib, Afatinib + Oxaliplatin, and Ulixertinib + Olaparib.

## Limitations

- No wet lab validation of predicted efficacious drug combinations or repositioned drugs has been performed yet.
- The study is limited in prioritizing synergy-based drug combinations, so literature corroboration reflects reported use rather than validated synergy.
- In silico predicted patient-specific drug cocktail sensitivity requires further evaluation with tumor-on-chip models, and lengthy regulatory processes for novel combinations can delay clinical deployment.

## Evidence
