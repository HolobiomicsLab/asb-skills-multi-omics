---
name: protein-complex-prior-probability-computation
description: 'Use when preparing Protrec2 inputs: you must convert curated complex
  records (updated_tissue_complex_*.csv) into per-tissue prior probabilities across
  the 27 supported tissues before Bayesian recovery.'
when_to_use_negative:
- You only have raw MS expression data and no complex catalogue — this skill scores
  curated complex records, not peak/intensity data
- You need protein-level (not complex-level) presence probabilities — those come from
  the later Bayesian recovery stage, not from these tissue priors
- Your tissue of interest is not among the 27 supported types (e.g. bile, bladder,
  ... uterine list in the README)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Protrec2
  role: Framework whose tissue-annotation stage computes the tissue-specific complex
    priors; ships the updated_tissue_complex_*.csv files and GO.tsv
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Used with pandas to load updated_tissue_complex_*.csv and compute/average
    per-tissue relevance and GO coherence scores
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:39178621bf8df15b3a3565fe2f97a111f62919f53ef27678bcdd7344f43ac3b9
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# protein-complex-prior-probability-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns each protein complex a tissue-specific relevance probability (1 = explicit tissue support, 0.5 = ambiguous/unspecified, 0 = explicitly non-relevant), with GO shared-term coherence as fallback, yielding the tissue-specific complex priors that Protrec2 feeds into Bayesian protein recovery.

## When to use

Use when preparing Protrec2 inputs: you must convert curated complex records (updated_tissue_complex_*.csv) into per-tissue prior probabilities across the 27 supported tissues before Bayesian recovery.

## When NOT to use

- You only have raw MS expression data and no complex catalogue — this skill scores curated complex records, not peak/intensity data
- You need protein-level (not complex-level) presence probabilities — those come from the later Bayesian recovery stage, not from these tissue priors
- Your tissue of interest is not among the 27 supported types (e.g. bile, bladder, ... uterine list in the README)

## Inputs

- data/updated_tissue_complex_*.csv (built-in tissue-specific complex records)
- data/GO.tsv (GO term associations for functional coherence)
- Tissue name selection (one of the 27 supported tissues, e.g. --tissue lung)

## Outputs

- Tissue-specific complex prior probability table P_complex_initial(c): one finite probability per complex per tissue (27 tissue columns)
- Intermediate per-tissue relevance scores (1.0 / 0.5 / 0.0) and GO coherence fallback scores

## How to apply

Load the built-in tissue-specific complex files (data/updated_tissue_complex_*.csv) with Python/pandas and, for each complex record, score tissue relevance as 1.0 when annotations explicitly support the tissue (e.g. lung, breast, brain), 0.0 when explicitly non-relevant, and 0.5 when ambiguous or unspecified. For complexes lacking direct tissue annotation, compute a GO shared-term functional coherence score (using data/GO.tsv) as the fallback relevance estimate. Average the direct annotation scores and the GO coherence fallback per complex and per tissue to produce P_complex_initial(c) for each of the 27 tissues. These priors form one of the three evidence layers—prior detection probability, cocomplex structural support, and tissue-specific complex confidence—that Protrec2's Bayesian inference integrates downstream.

## Related tools

- **Protrec2** (Framework whose tissue-annotation stage computes the tissue-specific complex priors; ships the updated_tissue_complex_*.csv files and GO.tsv) — https://github.com/miaomiao6606/Protrec2
- **Python** (Used with pandas to load updated_tissue_complex_*.csv and compute/average per-tissue relevance and GO coherence scores)

## Examples

```
import pandas as pd; tc = pd.read_csv('data/updated_tissue_complex_0.csv')
```

## Evaluation signals

- The output prior table covers all 27 tissue types matching the README's tissue list (bile through uterine)
- Every complex-tissue pair contains a finite probability; no NaNs in the prior CSV
- Spot-check annotated complexes: explicitly tissue-supported records score 1.0, ambiguous 0.5, explicitly non-relevant 0.0
- Complexes lacking direct annotation fall back to a computed GO shared-term coherence score rather than being dropped
- Priors feed the later Bayesian stage consistently with the three-layer evidence scheme (prior detection, cocomplex support, tissue confidence)

## Limitations

- Tissue-specific annotations were derived from population-level data and might not fully reflect interpatient heterogeneity
- The complex repository remains incomplete for novel or condition-specific complexes, so priors for such complexes rely solely on the GO coherence fallback
- Scoring is coarse (1.0 / 0.5 / 0.0) and binary in spirit; abundance, modification status, and subcellular localization are not captured

## Evidence

- [readme] Assign tissue relevance scores: 1.0: Explicit tissue support, 0.5: Ambiguous/unspecified, 0.0: Explicitly non-relevant: [evidence span withheld — anchor status: external_doc]
- [readme] GO-based functional coherence scoring is a key feature of Protrec2: [evidence span withheld — anchor status: external_doc]
