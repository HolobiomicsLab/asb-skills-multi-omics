---
name: go-shared-term-coherence-scoring
description: Use when building tissue-specific complex priors (updated_tissue_complex_*.csv)
  for Protrec2 and a complex record has no explicit tissue annotation, so GO-based
  functional coherence must stand in for a direct 1/0/0.5 relevance score.
when_to_use_negative:
- When the complex already carries explicit tissue annotation in the record — the
  direct 1/0/0.5 score takes precedence and the GO fallback is not needed
- When the goal is final protein-level recovery probabilities rather than tissue-specific
  complex priors — downstream Bayesian inference, not GO coherence scoring, produces
  those
- When working outside the 27 supported tissue types, since the priors and tissue-name
  sheet only cover the built-in tissue list
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0089
tools:
- name: Protrec2
  role: Framework whose tissue-annotation stage computes GO shared-term coherence
    as the fallback tissue relevance for unannotated complexes and integrates it into
    tissue-specific priors
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Used (e.g. with pandas) to load updated_tissue_complex_*.csv, compute relevance
    and GO coherence scores, average them per complex per tissue, and write the prior
    CSV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:42fab35931a297d3e596e87d364f2822deb631841d9ed30c57641d63bda520db
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# GO shared-term coherence scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

In Protrec2's tissue-annotation stage, Gene Ontology shared-term coherence is computed as the fallback tissue-relevance estimate for protein complexes lacking direct tissue annotation, and is averaged with direct annotations (1/0/0.5 relevance scores) to produce P_complex_initial(c) for each of the 27 tissue types.

## When to use

Use when building tissue-specific complex priors (updated_tissue_complex_*.csv) for Protrec2 and a complex record has no explicit tissue annotation, so GO-based functional coherence must stand in for a direct 1/0/0.5 relevance score.

## When NOT to use

- When the complex already carries explicit tissue annotation in the record — the direct 1/0/0.5 score takes precedence and the GO fallback is not needed
- When the goal is final protein-level recovery probabilities rather than tissue-specific complex priors — downstream Bayesian inference, not GO coherence scoring, produces those
- When working outside the 27 supported tissue types, since the priors and tissue-name sheet only cover the built-in tissue list

## Inputs

- updated_tissue_complex_*.csv tissue-specific complex records (built into Protrec2 under data/)
- data/GO.tsv Gene Ontology term associations for functional coherence scoring
- Tissue-name list of the 27 supported tissues (lung, breast, brain, etc.)

## Outputs

- Per-complex, per-tissue relevance scores (1.0 / 0.5 / 0.0 with GO coherence fallback)
- P_complex_initial(c): tissue-specific complex prior probability table as CSV, one probability per complex-tissue pair

## How to apply

Load the built-in updated_tissue_complex_*.csv records with Python/pandas and assign each complex a per-tissue relevance of 1.0 (explicit support), 0.0 (explicitly non-relevant), or 0.5 (ambiguous/unspecified) across the 27 tissue types (lung, breast, brain, etc.). For complexes lacking direct annotation, read GO term associations from data/GO.tsv and score functional coherence from GO terms shared among the complex's member proteins, using that coherence as the fallback tissue-relevance estimate. Average the direct annotation scores and the GO coherence fallback per complex and per tissue to obtain P_complex_initial(c), then write the resulting table as a CSV with one probability per complex per tissue. These priors are one of three evidence layers Protrec2 later integrates via Bayesian inference (prior detection probability, cocomplex structural support, tissue-specific complex confidence).

## Related tools

- **Protrec2** (Framework whose tissue-annotation stage computes GO shared-term coherence as the fallback tissue relevance for unannotated complexes and integrates it into tissue-specific priors) — https://github.com/miaomiao6606/Protrec2
- **Python** (Used (e.g. with pandas) to load updated_tissue_complex_*.csv, compute relevance and GO coherence scores, average them per complex per tissue, and write the prior CSV)

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- Output table covers all 27 tissue types from the provided tissue-name sheet (lung, breast, brain, etc.)
- Every complex-tissue pair has a finite prior probability between 0 and 1 in the CSV table
- Complexes with explicit tissue annotation show relevance values restricted to {1.0, 0.5, 0.0}; only complexes lacking annotation derive values from GO coherence fallback
- Tissue priors feed Protrec2's Bayesian inference as the third evidence layer (alongside prior detection probability and cocomplex structural support), so downstream runs with --tissue lung vs. other tissues should consume different prior values

## Limitations

- Tissue-specific annotations are derived from population-level data and may not reflect interpatient heterogeneity
- GO coherence is only a fallback: where record annotations are ambiguous or unspecified the score is 0.5, and coherence substitutes only when no direct annotation exists
- Protrec2's complex repository (CORUM, Complex Portal, Reactome, STRING-expanded) is incomplete for novel or condition-specific complexes, leaving no members to score coherence for

## Evidence

- [readme] GO data source for coherence scoring: [evidence span withheld — anchor status: not_found]
