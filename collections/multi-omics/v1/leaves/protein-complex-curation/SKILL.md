---
name: protein-complex-curation
description: Use when building or expanding a protein complex reference panel from
  heterogeneous sources (CORUM, STRING, Complex Portal, Reactome) for network-based
  protein recovery or co-complex analyses.
when_to_use_negative:
- The analysis requires protein abundance, modification status, or subcellular localization
  — the resulting panel supports binary presence/absence inference only
- 'The target complexes are novel or condition-specific: the curated repository is
  acknowledged to be incomplete for such complexes'
- Complexes smaller than 5 proteins are the focus — the size filter (≥5 proteins)
  excludes them at both the CORUM and Complex Portal/Reactome stages
edam_operation: http://edamontology.org/operation_3561
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0602
tools:
- name: Protrec2
  role: Framework whose complex_data.py module loads and processes the curated complex
    panel (CORUM, Complex Portal, Reactome, STRING expansion)
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Implementation language for complex generation, identifier validation, merging,
    and deduplication steps
  repo: https://github.com/miaomiao6606/Protrec2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:3ea0263f43d06bbec724496776a7b3561c14b8f0ee4c8325ed945c38c0a09bf9
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
  edam_operation: http://edamontology.org/operation_3561
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# protein-complex-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a comprehensive, deduplicated protein complex panel (3,885 complexes for Protrec2) by merging curated CORUM complexes with STRING-based all-or-nothing expansion, Complex Portal, and Reactome groupings, applying size and identifier-validity filters.

## When to use

Use when building or expanding a protein complex reference panel from heterogeneous sources (CORUM, STRING, Complex Portal, Reactome) for network-based protein recovery or co-complex analyses.

## When NOT to use

- The analysis requires protein abundance, modification status, or subcellular localization — the resulting panel supports binary presence/absence inference only
- The target complexes are novel or condition-specific: the curated repository is acknowledged to be incomplete for such complexes
- Complexes smaller than 5 proteins are the focus — the size filter (≥5 proteins) excludes them at both the CORUM and Complex Portal/Reactome stages

## Inputs

- CORUM complex database (e.g. human_complexes_2022.txt)
- STRING protein-protein interaction scores
- Complex Portal protein groupings
- Reactome pathway-derived protein groupings
- UniProt ID validation reference

## Outputs

- protein_complex_panel.csv (3,885 deduplicated protein complexes)
- Intermediate staged counts: 1,592 CORUM complexes -> 1,731 STRING-expanded complexes -> 3,885 merged panel

## How to apply

Load the CORUM complex database and retain only complexes with ≥5 proteins (1,592 complexes form the base panel). Expand each complex using STRING protein-protein interactions with an all-or-nothing rule: a candidate protein is admitted only if its combined interaction score with ALL existing complex members exceeds 780, growing the panel to 1,731 complexes. Parse Complex Portal and Reactome-derived protein groupings, keep those with ≥5 proteins, and admit them only if ≥80% of their protein identifiers are valid UniProt IDs. Merge all sources, normalize protein identifier formatting across complexes, and deduplicate complexes with identical protein membership, yielding a final panel of 3,885 complexes written as protein_complex_panel.csv. The all-or-nothing expansion rule prevents partial, biologically implausible complex inflation by requiring unanimous high-confidence STRING support.

## Related tools

- **Protrec2** (Framework whose complex_data.py module loads and processes the curated complex panel (CORUM, Complex Portal, Reactome, STRING expansion)) — https://github.com/miaomiao6606/Protrec2
- **Python** (Implementation language for complex generation, identifier validation, merging, and deduplication steps) — https://github.com/miaomiao6606/Protrec2

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- Final panel exists, is machine-readable, and its complex count equals 3,885
- Staged growth matches the reported progression 1,592 (CORUM ≥5 proteins) -> 1,731 (STRING-expanded) -> 3,885 (merged panel)
- Every admitted Complex Portal/Reactome grouping has ≥80% valid UniProt IDs
- No duplicate complexes remain — deduplication by identical protein membership yields unique complexes
- Each protein identifier is consistently formatted across all complexes in the panel

## Limitations

- The protein complex repository remains incomplete for novel or condition-specific complexes
- Expansion depends on STRING confidence (>780), so complexes with sparser high-confidence interaction data cannot be expanded
- Protein identifiers must resolve to UniProt IDs; groupings below the 80% validity threshold are discarded

## Evidence

- [results] During complex generation, a candidate protein is added to an existing CORUM complex only if it has high-confidence STRING interactions (score >780) with ALL proteins in the original complex: [evidence span withheld — anchor status: not_found]
- [results] Load the CORUM complex database and retain complexes with >=5 proteins to form the base panel (1592 complexes): [evidence span withheld — anchor status: not_found]
- [results] Validate protein identifiers within each Complex Portal/Reactome grouping, requiring >=80% valid UniProt IDs for admission: [evidence span withheld — anchor status: not_found]
- [readme] STRING-expanded complexes (confidence > 780); Total: 3,885 unique protein complexes: [evidence span withheld — anchor status: not_found]
- [readme] Filter for biological relevance (>80% valid UniProt IDs): [evidence span withheld — anchor status: external_doc]
