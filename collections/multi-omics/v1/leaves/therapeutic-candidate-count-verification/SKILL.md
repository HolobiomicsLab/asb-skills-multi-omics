---
name: therapeutic-candidate-count-verification
description: Use when reproducing or auditing a categorical breakdown of predicted
  drugs/drug combinations (e.g., CanSeer LUSC Fig. 6 outputs) where each category
  count must equal a reported value.
when_to_use_negative:
- The drug/combination lists are not from CanSeer's LUSC case study (Supplementary
  Data 15), so the expected category counts (4, 2, 26, 20, 97) do not apply as verification
  targets.
- The goal is to rank or prioritize drug combinations by predicted synergy — CanSeer's
  study explicitly 'remains limited in prioritizing synergy-based drug combinations',
  so this count-verification skill cannot support synergy ranking.
- Input is raw patient omics (mutations/RNA-seq) rather than already-predicted treatment
  lists; the upstream CanSeer modeling workflow must run first.
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_3379
- http://edamontology.org/topic_0203
tools:
- name: CanSeer
  role: Generates the predicted single-drug and drug-combination lists, FDA-approved
    repositioning candidates, and novel combination outputs that are classified and
    counted in this verification step
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:0dca5604ca90b9cb9fbc454184e1d4713da928c9383ca1933ade7507d5a87d38
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
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_3379
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# therapeutic-candidate-count-verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A verification skill for reproducing the per-category classification counts of CanSeer's predicted LUSC treatments — confirming that annotated mono-targeted drugs, FDA-approved combinations, literature-corroborated entries, repositioning candidates, and novel drug combinations sum to the reported totals (4, 2, 26, 20, 97).

## When to use

Use when reproducing or auditing a categorical breakdown of predicted drugs/drug combinations (e.g., CanSeer LUSC Fig. 6 outputs) where each category count must equal a reported value.

## When NOT to use

- The drug/combination lists are not from CanSeer's LUSC case study (Supplementary Data 15), so the expected category counts (4, 2, 26, 20, 97) do not apply as verification targets.
- The goal is to rank or prioritize drug combinations by predicted synergy — CanSeer's study explicitly 'remains limited in prioritizing synergy-based drug combinations', so this count-verification skill cannot support synergy ranking.
- Input is raw patient omics (mutations/RNA-seq) rather than already-predicted treatment lists; the upstream CanSeer modeling workflow must run first.

## Inputs

- Predicted single-drug list from CanSeer (Supplementary Data 15)
- Predicted drug-combination list from CanSeer (Supplementary Data 15)
- FDA approval status annotations for NSCLC/LUSC drugs
- Prior-literature references supporting existing drug/combination use

## Outputs

- drug_classification.csv — categorical table with one label per drug/combination (FDA-approved mono, FDA-approved combination, literature-corroborated, repositioning candidate, novel combination)
- Per-category count totals: 4 mono-targeted FDA-approved drugs, 2 FDA-approved combinations, 26 literature-corroborated, 20 repositioning candidates, 97 novel combinations

## How to apply

Load the predicted single-drug and drug-combination lists from Supplementary Data 15 (the source of the Fig. 6 categorization). Annotate each entry with its FDA approval status, flagging the four FDA-approved mono-targeted drugs (Afatinib, Osimertinib, Dabrafenib, Trametinib) and the two FDA-approved combinations (Osimertinib + Oxaliplatin, Dabrafenib + Trametinib). Cross-reference remaining entries against prior literature to label the 26 literature-corroborated drugs/combinations, then label the 20 FDA-approved repositioning candidates per CanSeer's repositioning output. Assign the 97 remaining drug combinations lacking prior support (e.g., Afuresertib + Palbociclib, Dinaciclib + Trametinib, Afatinib + Oxaliplatin, Ulixertinib + Olaparib) as novel combinations. Verify each per-category count matches the reported values (4, 2, 26, 20, 97) and write the categorized table to drug_classification.csv.

## Related tools

- **CanSeer** (Generates the predicted single-drug and drug-combination lists, FDA-approved repositioning candidates, and novel combination outputs that are classified and counted in this verification step) — github.com/BIRL/CanSeer

## Examples

```
import pandas as pd; df = pd.read_csv('SupplementaryData15.csv'); counts = df['category'].value_counts(); assert counts[['FDA_mono','FDA_combo','literature','repositioning','novel']].tolist() == [4,2,26,20,97]; df.to_csv('drug_classification.csv')
```

## Evaluation signals

- Count of FDA-approved mono-targeted drugs equals 4 (Afatinib, Osimertinib, Dabrafenib, Trametinib).
- Count of FDA-approved combinations equals 2 (Osimertinib + Oxaliplatin, Dabrafenib + Trametinib).
- Literature-corroborated category totals 26 entries and repositioning candidates total 20.
- Novel-combination category totals 97, with named examples (Afuresertib + Palbociclib, Dinaciclib + Trametinib, Afatinib + Oxaliplatin, Ulixertinib + Olaparib) assigned to it.
- drug_classification.csv exists and every entry carries exactly one of the five category labels with no unclassified rows.

## Limitations

- Category assignment for the 26 literature-corroborated and 97 novel combinations depends on manual cross-referencing against prior literature, so labels are subject to curation bias.
- FDA approval status is annotated for non-small cell lung cancer context; approvals in other indications would be misclassified if not checked against NSCLC specifically.
- Counts reflect CanSeer's LUSC case study outputs and do not transfer to other cancer types or updated FDA approval databases.

## Evidence

- [other] Load the predicted single-drug and drug-combination lists from Supplementary Data 15 (the source of the Fig. 6 categorization): [evidence span withheld — anchor status: not_found]
- [other] Verify the per-category counts equal the reported values (4, 2, 26, 20, 97) and write the categorized table to drug_classification.csv: [evidence span withheld — anchor status: not_found]
