---
name: string-interaction-network-filtering
description: Use when building or enlarging a protein complex panel from CORUM/Complex
  Portal/Reactome sources and you need high-confidence STRING-based expansion with
  strict all-member co-interaction evidence.
when_to_use_negative:
- When STRING interactions alone (without co-complex membership from CORUM/Complex
  Portal/Reactome) are the intended evidence — this skill uses STRING only to expand
  existing curated complexes, not as a standalone network
- When sensitivity for low-scoring complexes matters more than specificity — the all-or-nothing
  rule at score >780 intentionally prevents confident recovery of proteins in lower-scoring
  complexes (a known PROTREC limitation)
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0602
tools:
- name: Protrec2
  role: 'Implements the complex generation pipeline: CORUM loading, STRING expansion
    at score >780, Complex Portal/Reactome integration, identifier validation, and
    deduplication to produce protein_complex_panel.csv'
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Execution environment for the Protrec2 complex generation workflow (main.py
    entry point, requirements.txt dependencies)
  repo: https://github.com/miaomiao6606/Protrec2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:99d6f1ddfa3d4dd0fc64ea9422d4add22c1ec7cb87246fa67d76ded2a18a417d
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
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# string-interaction-network-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Expands curated protein complexes (e.g. CORUM complexes of size >=5) using STRING protein-protein interaction scores, admitting a candidate protein only if its combined interaction score with ALL existing complex members exceeds 780 (all-or-nothing rule). This grows the complex panel while enforcing strict co-complex evidence before expansion.

## When to use

Use when building or enlarging a protein complex panel from CORUM/Complex Portal/Reactome sources and you need high-confidence STRING-based expansion with strict all-member co-interaction evidence.

## When NOT to use

- When STRING interactions alone (without co-complex membership from CORUM/Complex Portal/Reactome) are the intended evidence — this skill uses STRING only to expand existing curated complexes, not as a standalone network
- When sensitivity for low-scoring complexes matters more than specificity — the all-or-nothing rule at score >780 intentionally prevents confident recovery of proteins in lower-scoring complexes (a known PROTREC limitation)

## Inputs

- CORUM complex database (human complexes, size >=5 proteins)
- STRING protein-protein interaction scores (combined score, cutoff >780)
- Complex Portal protein groupings (EMBL-EBI validated complexes)
- Reactome pathway-derived protein groupings

## Outputs

- protein_complex_panel.csv containing 3885 deduplicated protein complexes
- Expanded CORUM complexes (1592 base -> 1731 after STRING expansion)

## How to apply

Start from the CORUM database and retain only complexes with >=5 proteins, yielding a base panel of 1592 complexes. For each complex, admit a STRING-interacting candidate protein only if its combined interaction score exceeds 780 with ALL proteins in the original complex — the all-or-nothing rule — which grows the panel to 1731 complexes. In parallel, parse Complex Portal and Reactome groupings of >=5 proteins, validate that >=80% of identifiers are valid UniProt IDs, merge all sources, normalize identifier formatting, and deduplicate complexes with identical protein membership to reach the final 3885-complex panel written as protein_complex_panel.csv. The all-or-nothing rule is the key decision point: it prevents single weak interactions from pulling in spurious proteins, at the cost of reduced sensitivity for lower-scoring complexes.

## Related tools

- **Protrec2** (Implements the complex generation pipeline: CORUM loading, STRING expansion at score >780, Complex Portal/Reactome integration, identifier validation, and deduplication to produce protein_complex_panel.csv) — https://github.com/miaomiao6606/Protrec2
- **Python** (Execution environment for the Protrec2 complex generation workflow (main.py entry point, requirements.txt dependencies)) — https://github.com/miaomiao6606/Protrec2

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- Staged complex counts match the reported growth: 1592 CORUM base complexes -> 1731 after STRING expansion -> 3885 final merged panel
- protein_complex_panel.csv exists, is machine-readable (CSV), and contains exactly 3885 complexes
- Every expanded complex retains only candidate proteins whose STRING combined score exceeds 780 with ALL members of the original complex
- All Complex Portal/Reactome-derived complexes contain >=5 proteins with >=80% valid UniProt IDs
- No duplicate complexes with identical protein membership remain in the final panel

## Limitations

- CORUM alone contains only 694 human protein complexes of size >=5, limiting coverage of the broader human complexome — hence the need for expansion and additional sources
- The all-or-nothing STRING expansion rule enforces high specificity but reduces sensitivity, preventing confident identification of proteins in lower-scoring complexes
- The merged protein complex repository, while comprehensive (3885 complexes), remains incomplete for novel or condition-specific complexes

## Evidence

- [full_text] All-or-nothing STRING expansion rule with score cutoff 780: [evidence span withheld — anchor status: not_found]
- [readme] UniProt ID validation for Complex Portal/Reactome groupings: [evidence span withheld — anchor status: external_doc]
