---
name: multi-database-complex-integration
description: Use when a complex-based recovery or enrichment method needs a broader
  complex prior than CORUM alone (only 694 human complexes of size >=5), and CORUM,
  STRING, Complex Portal, and Reactome sources are available to merge and deduplicate.
when_to_use_negative:
- The analysis requires only manually curated CORUM complexes (e.g., as curated ground
  truth) — STRING expansion injects inferred members into curated complexes, altering
  membership.
- Complexes smaller than 5 proteins are needed — the workflow explicitly retains only
  complexes/groupings with >=5 proteins.
- Source groupings cannot be mapped to UniProt identifiers (fewer than 80% valid IDs),
  so they would be rejected during identifier validation.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3366
tools:
- name: Protrec2
  role: 'Implements the complex-generation workflow: loads CORUM (size >=5), expands
    with STRING (score >780, all-or-nothing), integrates Complex Portal and Reactome,
    filters for >=80% valid UniProt IDs, and writes the 3,885-complex protein_complex_panel.csv'
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Scripting language for parsing the four sources, UniProt ID validation, identifier
    normalization, merging, and deduplication
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:f2c07e7e0a821eca418713b25f782f93534fd4b39603e1f9688bb6c65a26c43b
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# multi-database-complex-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs one comprehensive protein complex panel by merging CORUM complexes (size >=5), STRING all-or-nothing expansion (combined score >780 with every member), and Complex Portal/Reactome groupings (>=80% valid UniProt IDs), then deduplicates and normalizes identifiers. In Protrec2 this grew 1,592 curated CORUM base complexes to a final 3,885-complex panel, overcoming CORUM's limited coverage of the broader human complexome.

## When to use

Use when a complex-based recovery or enrichment method needs a broader complex prior than CORUM alone (only 694 human complexes of size >=5), and CORUM, STRING, Complex Portal, and Reactome sources are available to merge and deduplicate.

## When NOT to use

- The analysis requires only manually curated CORUM complexes (e.g., as curated ground truth) — STRING expansion injects inferred members into curated complexes, altering membership.
- Complexes smaller than 5 proteins are needed — the workflow explicitly retains only complexes/groupings with >=5 proteins.
- Source groupings cannot be mapped to UniProt identifiers (fewer than 80% valid IDs), so they would be rejected during identifier validation.

## Inputs

- CORUM complex database file (e.g., data/human_complexes_2022.txt), filtered to complexes with >=5 proteins
- STRING protein-protein interaction network with combined confidence scores per protein pair
- Complex Portal (EMBL-EBI) complex/protein group listings
- Reactome-derived protein groupings
- Reference set of valid UniProt identifiers for ID validation

## Outputs

- protein_complex_panel.csv: deduplicated, identifier-normalized panel of 3,885 unique protein complexes (each >=5 proteins)
- Staged intermediate counts for validation: 1,592 CORUM base complexes -> 1,731 after STRING expansion -> 3,885 final merged panel

## How to apply

Load the CORUM complex database and retain only complexes with >=5 proteins to form the base panel (1,592 complexes). Expand each complex using STRING protein-protein interactions under an all-or-nothing rule: a candidate protein is admitted only if its combined interaction score with ALL existing complex members exceeds 780, growing the panel to 1,731 complexes. Parse Complex Portal and Reactome-derived protein groupings, keep those containing >=5 proteins, and admit only groupings with >=80% valid UniProt IDs (the README's 'biological relevance' filter). Merge all sources, normalize protein identifier formatting across complexes, and deduplicate complexes with identical protein membership. Write the final panel of 3,885 complexes as protein_complex_panel.csv, which then feeds Protrec2's tissue-specific Bayesian recovery (tissue priors from updated_tissue_complex_*.csv). The all-or-nothing expansion rule is the key design decision: it preserves complex integrity during expansion while compensating for CORUM's limited coverage of the human complexome.

## Related tools

- **Protrec2** (Implements the complex-generation workflow: loads CORUM (size >=5), expands with STRING (score >780, all-or-nothing), integrates Complex Portal and Reactome, filters for >=80% valid UniProt IDs, and writes the 3,885-complex protein_complex_panel.csv) — https://github.com/miaomiao6606/Protrec2
- **Python** (Scripting language for parsing the four sources, UniProt ID validation, identifier normalization, merging, and deduplication)

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- Final panel complex count equals 3,885, consistent with the reported staged growth 1,592 -> 1,731 -> 3,885.
- protein_complex_panel.csv exists and is machine-readable (parses as CSV).
- No two complexes in the panel share identical protein membership (deduplication invariant).
- Every retained CORUM base complex has >=5 proteins, and every admitted Complex Portal/Reactome grouping has >=5 proteins with >=80% valid UniProt IDs.
- Every STRING-expanded member has a combined interaction score >780 with ALL proteins of the original complex, and identifier formatting is uniform across complexes.

## Limitations

- Even the integrated 3,885-complex repository remains incomplete for novel or condition-specific complexes, so panel coverage bounds downstream recovery.
- The all-or-nothing STRING rule (combined score >780 with every original member) is conservative; candidates supported by only a subset of complex members are excluded.
- Complex Portal/Reactome groupings failing the >=80% valid UniProt ID check are discarded wholesale, so identifier-mapping quality directly gates panel composition.

## Evidence

- [readme] Load curated complexes from CORUM (size ≥ 5 proteins): [evidence span withheld — anchor status: external_doc]
- [readme] STRING-expanded complexes (confidence > 780): [evidence span withheld — anchor status: external_doc]
- [readme] Filter for biological relevance (>80% valid UniProt IDs): [evidence span withheld — anchor status: external_doc]
- [other] admit a candidate protein only if its combined interaction score with ALL existing complex members exceeds 780 (all-or-nothing rule): [evidence span withheld — anchor status: not_found]
- [other] Parse Complex Portal and Reactome-derived protein groupings and keep those containing >=5 proteins: [evidence span withheld — anchor status: not_found]
- [other] Write the final panel (3885 complexes) as protein_complex_panel.csv: [evidence span withheld — anchor status: not_found]
