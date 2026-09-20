---
name: drug-combination-categorization
description: Use when you have a list of predicted single drugs and drug combinations
  from CanSeer (e.g., from Supplementary Data) and need to stratify them by FDA approval
  status, prior literature support, repositioning status, or novelty.
when_to_use_negative:
- When the input is not a CanSeer-style predicted drug/combination list but raw omics
  data — use CanSeer's model-building and drug-scoring workflow first
- When synergy-based prioritization of combinations is the goal — the article notes
  the study 'remains limited in prioritizing synergy-based drug combinations', so
  this categorization does not rank synergy
- When in vitro/wet-lab validation of efficacy is needed — categorization is purely
  in silico and literature-based
tools:
- name: CanSeer
  role: Source of the predicted single-drug and drug-combination lists and the FDA-approved
    drug repositioning output that are categorized
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:313597a4db30ef2eeceeaab36471c1227f882355609dbe539fdc3c56956fb6de
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# drug-combination-categorization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Categorizes CanSeer-predicted mono-targeted drugs and drug combinations for a cancer type (e.g., LUSC) into approval-status and evidence tiers: FDA-approved drugs, FDA-approved combinations, literature-corroborated entries, FDA-approved repositioning candidates, and novel combinations.

## When to use

Use when you have a list of predicted single drugs and drug combinations from CanSeer (e.g., from Supplementary Data) and need to stratify them by FDA approval status, prior literature support, repositioning status, or novelty.

## When NOT to use

- When the input is not a CanSeer-style predicted drug/combination list but raw omics data — use CanSeer's model-building and drug-scoring workflow first
- When synergy-based prioritization of combinations is the goal — the article notes the study 'remains limited in prioritizing synergy-based drug combinations', so this categorization does not rank synergy
- When in vitro/wet-lab validation of efficacy is needed — categorization is purely in silico and literature-based

## Inputs

- Predicted single-drug list from CanSeer output (e.g., Supplementary Data 15)
- Predicted drug-combination list from CanSeer output (e.g., Supplementary Data 15)
- CanSeer FDA-approved drug repositioning output
- Prior literature / clinical evidence records for cross-referencing drug and combination support

## Outputs

- Categorized drug/combination classification table (e.g., drug_classification.csv) with labels: FDA-approved mono-targeted drugs, FDA-approved combinations, literature-corroborated entries, repositioning candidates, and novel combinations
- Per-category counts (e.g., 4 FDA-approved mono drugs, 2 FDA-approved combinations, 26 literature-corroborated, 20 repositioning candidates, 97 novel combinations in the LUSC case)

## How to apply

Load the predicted single-drug and drug-combination lists from the relevant supplementary dataset (CanSeer's LUSC case study used Supplementary Data 15, the source of the Fig. 6 categorization). Annotate each entry with its FDA approval status, first flagging mono-targeted drugs that are FDA-approved for the indication (Afatinib, Osimertinib, Dabrafenib, Trametinib in the LUSC case) and FDA-approved combinations (e.g., Osimertinib + Oxaliplatin, Dabrafenib + Trametinib). Cross-reference remaining entries against prior literature to label literature-corroborated drugs/combinations (26 in the LUSC case), then label repositioning candidates from CanSeer's FDA-approved drug repositioning output (20 candidates). Label remaining combinations lacking prior support (e.g., Afuresertib + Palbociclib, Dinaciclib + Trametinib, Afatinib + Oxaliplatin, Ulixertinib + Olaparib) as novel combinations (97 in the LUSC case). Finally verify per-category counts and write the categorized table to a machine-readable output such as drug_classification.csv.

## Related tools

- **CanSeer** (Source of the predicted single-drug and drug-combination lists and the FDA-approved drug repositioning output that are categorized) — github.com/BIRL/CanSeer

## Examples

```
df.to_csv('drug_classification.csv')  # write CanSeer LUSC drug/combination categories (4 FDA-approved, 2 FDA-approved combos, 26 literature, 20 repositioned, 97 novel)
```

## Evaluation signals

- Per-category counts match the reported values (e.g., 4 FDA-approved mono-targeted drugs, 2 FDA-approved combinations, 26 literature-corroborated, 20 repositioning candidates, 97 novel combinations for LUSC)
- Known FDA-approved LUSC drugs (Afatinib, Osimertinib, Dabrafenib, Trametinib) and combinations (Osimertinib + Oxaliplatin, Dabrafenib + Trametinib) land in the correct approval categories
- Novel-combination tier contains the expected exemplars such as Afuresertib + Palbociclib, Dinaciclib + Trametinib, Afatinib + Oxaliplatin, and Ulixertinib + Olaparib
- Every entry in the input list appears in exactly one category (no duplicates or unclassified rows)
- Output file (drug_classification.csv) parses cleanly and preserves the source categorization of Fig. 6 / Supplementary Data 15

## Limitations

- CanSeer's predictions are in silico; wet-lab validation of predicted efficacious drug combinations and repositioned drugs is still needed for clinical acceptability
- Patient-specific sensitivity to drug cocktails revealed by the in silico personalized models requires further evaluation with tumor-on-chip models
- The underlying study is limited in prioritizing synergy-based drug combinations
- Lengthy regulatory processes for novel drug combinations can delay clinical deployment of in silico predictions

## Evidence
