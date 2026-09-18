---
name: complex-panel-deduplication
description: Use when a network-based protein recovery method needs a comprehensive,
  deduplicated protein complex reference panel beyond CORUM's 694 human complexes
  (≥5 proteins), by merging CORUM, STRING-expanded, Complex Portal, and Reactome sources.
when_to_use_negative:
- You need per-protein recovery probabilities or Bayesian inference over the panel
  — this skill only constructs the complex reference panel, not the downstream Bayesian
  protein recovery step
- Your analysis requires complexes smaller than 5 proteins — the panel explicitly
  filters to complexes with >=5 proteins
- Your target proteome is non-human/non-mammalian — the base CORUM complexes and coverage
  rationale are human-specific (694 human complexes of size >=5)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0082
tools:
- name: Protrec2
  role: Framework whose Step 1 'Comprehensive Protein Complex Generation' module loads
    CORUM, expands with STRING, integrates Complex Portal/Reactome, and writes the
    complex panel
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Implementation language for complex loading, STRING expansion, identifier
    validation, merging, and deduplication of the panel
  repo: https://github.com/miaomiao6606/Protrec2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:54d78a0cf320e26bee42fb35026f8656ce944d8bbf01f1514e668e44a1742551
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/miaomiao6606/Protrec2
  repo_url: https://github.com/miaomiao6606/Protrec2
  tools:
  - Protrec2
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0082
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# complex-panel-deduplication

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs Protrec2's comprehensive protein complex reference panel by expanding CORUM complexes with STRING interactions (all-or-nothing, score >780), integrating Complex Portal and Reactome groupings, then merging, normalizing identifiers, and deduplicating to a final 3,885-complex protein_complex_panel.csv.

## When to use

Use when a network-based protein recovery method needs a comprehensive, deduplicated protein complex reference panel beyond CORUM's 694 human complexes (≥5 proteins), by merging CORUM, STRING-expanded, Complex Portal, and Reactome sources.

## When NOT to use

- You need per-protein recovery probabilities or Bayesian inference over the panel — this skill only constructs the complex reference panel, not the downstream Bayesian protein recovery step
- Your analysis requires complexes smaller than 5 proteins — the panel explicitly filters to complexes with >=5 proteins
- Your target proteome is non-human/non-mammalian — the base CORUM complexes and coverage rationale are human-specific (694 human complexes of size >=5)

## Inputs

- CORUM complex database (manually curated mammalian complexes, e.g. human_complexes_2022.txt)
- STRING protein-protein interaction scores (combined score, all-or-nothing admission cutoff >780)
- Complex Portal (EMBL-EBI validated) protein groupings
- Reactome pathway-derived protein groupings
- UniProt protein identifiers requiring format normalization/validation

## Outputs

- protein_complex_panel.csv (3,885 deduplicated protein complexes)
- Intermediate staged counts: 1592 CORUM base complexes -> 1731 STRING-expanded complexes -> 3885 final panel

## How to apply

Load the CORUM complex database and retain complexes with >=5 proteins as the base panel (1592 complexes), addressing CORUM's limited coverage. Expand each complex using STRING protein-protein interactions, admitting a candidate protein only if its combined interaction score with ALL existing members exceeds 780 (all-or-nothing rule), growing the panel to 1731 complexes. Parse Complex Portal and Reactome-derived protein groupings and keep those containing >=5 proteins, validating that >=80% of protein identifiers in each grouping are valid UniProt IDs. Merge all sources, normalize protein identifier formatting across complexes, and deduplicate complexes with identical protein membership. Write the final panel (3885 complexes) as protein_complex_panel.csv and verify it is machine-readable with the expected count and staged growth 1592 -> 1731 -> 3885.

## Related tools

- **Protrec2** (Framework whose Step 1 'Comprehensive Protein Complex Generation' module loads CORUM, expands with STRING, integrates Complex Portal/Reactome, and writes the complex panel) — https://github.com/miaomiao6606/Protrec2
- **Python** (Implementation language for complex loading, STRING expansion, identifier validation, merging, and deduplication of the panel) — https://github.com/miaomiao6606/Protrec2

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- protein_complex_panel.csv exists, is machine-readable (CSV parseable), and contains exactly 3885 complexes
- Staged growth matches the reported progression 1592 (CORUM base) -> 1731 (STRING-expanded) -> 3885 (merged panel)
- No two complexes in the final panel share identical protein membership (deduplication invariant)
- Every retained Complex Portal/Reactome grouping has >=80% valid UniProt identifiers and >=5 proteins
- Each expanded complex grew only via all-or-nothing STRING admission: every added protein has combined interaction score >780 with ALL original complex members

## Limitations

- The protein complex repository, while comprehensive (3885 complexes), remains incomplete for novel or condition-specific complexes
- CORUM's base coverage is limited (only 694 human protein complexes of size >=5), so panel breadth depends on STRING expansion and Complex Portal/Reactome supplementation
- The STRING >780 all-or-nothing expansion rule is stringent, so candidates lacking high-confidence interactions with every member are never admitted

## Evidence

- [intro] as CORUM only contains 694 human protein complexes above the size threshold (≥5 proteins), this limits its coverage of the broader human complexome and proteome: "as CORUM only contains 694 human protein complexes above the size threshold (≥5 proteins), this limits its coverage of the broader human complexome and proteome"
- [readme] Load curated complexes from CORUM (size ≥ 5 proteins): [evidence span withheld — anchor status: external_doc]
- [readme] STRING-expanded complexes (confidence > 780): [evidence span withheld — anchor status: external_doc]
- [readme] Filter for biological relevance (>80% valid UniProt IDs): [evidence span withheld — anchor status: external_doc]
- [readme] Total: 3,885 unique protein complexes: [evidence span withheld — anchor status: external_doc]
