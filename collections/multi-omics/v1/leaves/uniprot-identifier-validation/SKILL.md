---
name: uniprot-identifier-validation
description: Use when merging externally sourced protein groupings (Complex Portal,
  Reactome, CORUM, STRING expansions) into a unified panel and you must screen records
  whose protein identifiers may be malformed or non-UniProt before admission.
when_to_use_negative:
- When the source groupings are already curated UniProt-ID-based sets with known-valid
  membership (e.g. the CORUM >=5-protein base panel), re-validating adds no value
- When the input expression matrix uses gene symbols that the downstream method explicitly
  accepts — validation should not reject valid non-UniProt identifiers that the pipeline
  is designed to handle
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3557
tools:
- name: Protrec2
  role: Implements the complex-generation pipeline in which UniProt-ID validation
    (>=80% valid IDs) gates Complex Portal/Reactome groupings before panel assembly
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Scripting language used to parse complex sources, check identifier validity,
    normalize formatting, and write the final panel CSV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:c324a8af3d93da7c66afadfea31f9696bd77ca1064a0465674553bc8c88b5aac
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
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3557
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.0
n_claims: 4
---

# uniprot-identifier-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A quality-gate step that checks the fraction of valid UniProt identifiers within protein-grouping records (e.g. Complex Portal or Reactome-derived complexes) before admitting them into a downstream panel, as used in Protrec2's protein complex dataset construction.

## When to use

Use when merging externally sourced protein groupings (Complex Portal, Reactome, CORUM, STRING expansions) into a unified panel and you must screen records whose protein identifiers may be malformed or non-UniProt before admission.

## When NOT to use

- When the source groupings are already curated UniProt-ID-based sets with known-valid membership (e.g. the CORUM >=5-protein base panel), re-validating adds no value
- When the input expression matrix uses gene symbols that the downstream method explicitly accepts — validation should not reject valid non-UniProt identifiers that the pipeline is designed to handle

## Inputs

- Complex Portal protein groupings (parsed, >=5 proteins)
- Reactome-derived protein groupings (parsed, >=5 proteins)
- Expression matrix protein identifiers (UniProt IDs or gene symbols) from CSV input

## Outputs

- Admitted set of Complex Portal/Reactome groupings passing the >=80% valid-UniProt-ID criterion
- Identifier-normalized, deduplicated protein complex panel (protein_complex_panel.csv, 3,885 complexes)

## How to apply

After parsing protein groupings from Complex Portal and Reactome and retaining only those with >=5 proteins, validate the protein identifiers in each grouping: count how many members resolve to valid UniProt IDs and admit the grouping only if at least 80% of its identifiers are valid UniProt IDs. This acts as an admission filter before merging all sources (CORUM base panel, STRING-expanded complexes, Complex Portal, Reactome) into the final 3,885-complex panel. After validation, normalize protein identifier formatting across complexes and deduplicate complexes with identical protein membership. The same identifier-hygiene concern applies to user inputs: Protrec2's troubleshooting guidance requires expression-matrix protein IDs to be in UniProt format.

## Related tools

- **Protrec2** (Implements the complex-generation pipeline in which UniProt-ID validation (>=80% valid IDs) gates Complex Portal/Reactome groupings before panel assembly) — https://github.com/miaomiao6606/Protrec2
- **Python** (Scripting language used to parse complex sources, check identifier validity, normalize formatting, and write the final panel CSV)

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- Groupings with fewer than 80% valid UniProt IDs are excluded from the panel — spot-check rejected records to confirm the cutoff is applied
- All retained groupings contain >=5 proteins before and after validation
- Final protein_complex_panel.csv contains exactly 3,885 complexes, consistent with the staged growth 1592 (CORUM) -> 1731 (STRING-expanded) -> 3885 (merged)
- No duplicate complexes with identical protein membership remain in the output panel
- Expression-matrix protein identifiers flagged in troubleshooting are in UniProt format

## Limitations

- The >=80% UniProt-ID validity cutoff is a heuristic admission threshold; it may discard groupings with otherwise meaningful biology but partially malformed identifiers
- Validation operates on identifier format/validity only; it does not resolve ambiguous mappings between gene symbols and UniProt accessions
- The protein complex repository used downstream remains incomplete for novel or condition-specific complexes regardless of identifier quality

## Evidence

- [other] Validate protein identifiers within each Complex Portal/Reactome grouping, requiring >=80% valid UniProt IDs for admission.: [evidence span withheld — anchor status: not_found]
- [readme] Filter for biological relevance (>80% valid UniProt IDs): [evidence span withheld — anchor status: external_doc]
- [readme] Verify protein IDs are UniProt format: [evidence span withheld — anchor status: not_found]
- [other] Merge all sources, normalize protein identifier formatting across complexes, and deduplicate complexes with identical protein membership.: [evidence span withheld — anchor status: not_found]
