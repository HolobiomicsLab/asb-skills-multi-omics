---
name: corum-complex-tissue-relevance-scoring
description: Use when building tissue-specific complex prior probabilities (P_complex_initial)
  for Protrec2 — e.g. when curated complex records in updated_tissue_complex_.csv
  must be scored across the 27 supported tissues before Bayesian inference.
when_to_use_negative:
- Input is a raw expression matrix awaiting Protrec2 recovery (main.py handles priors
  internally) rather than complex record annotation
- The target tissue is not among the 27 supported types (bile, bladder, bone, brain,
  breast, cervical, colorectal, esophageal, gallbladder, gastric, head_and_neck, kidney,
  leukemia, liver, lung, lymphoma, myeloma, neuroblastoma, ovarian, pancreatic, prostate,
  rhabdoid_tumor, sarcoma, skin, testis, thyroid, uterine)
- The goal is scoring individual proteins rather than protein complexes — this skill
  operates at the complex level
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_1775
tools:
- name: Protrec2
  role: Framework whose tissue-annotation stage consumes the 1/0/0.5 + GO-coherence
    complex priors; provides built-in updated_tissue_complex_*.csv records
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Used (with pandas) to load complex records, apply the scoring scheme, average
    with GO coherence fallback, and write the prior CSV
- name: CORUM
  role: Source of curated mammalian protein complexes that receive the tissue relevance
    scores
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:d5b5075292f650b9a5edb4ef94d04fca7e4ab8ff71e3bf8b0e48e818d99a4081
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
  - CORUM
  - Protrec2
  - Python
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_1775
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# corum-complex-tissue-relevance-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns per-tissue relevance probabilities to protein complexes (e.g. CORUM-derived sets) using a 1/0/0.5 annotation scheme with a GO shared-term coherence fallback, producing tissue-specific complex priors for Protrec2's Bayesian recovery stage.

## When to use

Use when building tissue-specific complex prior probabilities (P_complex_initial) for Protrec2 — e.g. when curated complex records in updated_tissue_complex_*.csv must be scored across the 27 supported tissues before Bayesian inference.

## When NOT to use

- Input is a raw expression matrix awaiting Protrec2 recovery (main.py handles priors internally) rather than complex record annotation
- The target tissue is not among the 27 supported types (bile, bladder, bone, brain, breast, cervical, colorectal, esophageal, gallbladder, gastric, head_and_neck, kidney, leukemia, liver, lung, lymphoma, myeloma, neuroblastoma, ovarian, pancreatic, prostate, rhabdoid_tumor, sarcoma, skin, testis, thyroid, uterine)
- The goal is scoring individual proteins rather than protein complexes — this skill operates at the complex level

## Inputs

- Tissue-specific complex record CSVs (data/updated_tissue_complex_*.csv)
- Gene Ontology term associations (data/GO.tsv) for coherence fallback scoring
- Protein complex list (e.g. CORUM, Complex Portal, Reactome-derived, data/complex_list.csv)
- Tissue-name sheet enumerating the 27 supported tissue types

## Outputs

- Tissue-specific complex prior probability table (CSV, one P_complex_initial per complex per tissue)
- Per-complex, per-tissue relevance scores (1 / 0.5 / 0 or GO coherence fallback)

## How to apply

Load the built-in tissue-specific complex records (updated_tissue_complex_*.csv) with Python/pandas and, for each complex record, assign a relevance score per tissue: 1.0 when record annotations explicitly support the tissue, 0.0 when explicitly non-relevant, and 0.5 when evidence is ambiguous or unspecified. For complexes lacking direct tissue annotation, compute a GO shared-term coherence score (using data/GO.tsv associations) as the fallback tissue relevance estimate. Average the direct annotations and the GO coherence fallback per complex–tissue pair to produce P_complex_initial(c) for each of the 27 tissues (lung, breast, brain, etc.), then write the resulting priors as a CSV table with one probability per complex per tissue. These priors feed Protrec2's Bayesian inference, which integrates prior detection probability, cocomplex structural support, and tissue-specific complex confidence.

## Related tools

- **Protrec2** (Framework whose tissue-annotation stage consumes the 1/0/0.5 + GO-coherence complex priors; provides built-in updated_tissue_complex_*.csv records) — https://github.com/miaomiao6606/Protrec2
- **Python** (Used (with pandas) to load complex records, apply the scoring scheme, average with GO coherence fallback, and write the prior CSV)
- **CORUM** (Source of curated mammalian protein complexes that receive the tissue relevance scores)

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('data/updated_tissue_complex_0.csv'); print(df.columns)"
```

## Evaluation signals

- Output prior table covers all 27 tissue types listed in the provided tissue-name sheet
- Every complex–tissue pair has a finite probability in [0, 1]
- Scores follow the scheme: 1.0 for explicit tissue support, 0.0 for explicit non-relevance, 0.5 for ambiguous/unspecified evidence
- Complexes lacking direct annotation receive a GO shared-term coherence fallback rather than a default constant
- Priors feed downstream Bayesian recovery consistent with the README's Step 2 ('Integrate scores into tissue-specific priors')

## Limitations

- Tissue-specific annotations were derived from population-level data and might not fully reflect interpatient heterogeneity
- The protein complex repository (CORUM, Complex Portal, Reactome, STRING-expanded) remains incomplete for novel or condition-specific complexes
- Ambiguous or unspecified tissue evidence defaults to 0.5, which may dilute priors when annotation quality is poor

## Evidence

- [readme] 27 distinct tissue annotations in Protrec2: [evidence span withheld — anchor status: external_doc]
