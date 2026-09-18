---
name: lung-tissue-specific-complex-annotation
description: Use when a proteomics recovery task needs tissue-aware complex priors
  — e.g., running Protrec2 with --tissue lung on an expression matrix and requiring
  per-complex confidence values across 27 tissue types.
when_to_use_negative:
- Input is an expression matrix you want scored directly — this skill produces complex-level
  priors, not per-sample protein probabilities (run main.py recovery for that)
- Studying a tissue outside the 27 supported types (bile, bladder, bone, brain, breast,
  cervical, colorectal, esophageal, gallbladder, gastric, head_and_neck, kidney, leukemia,
  liver, lung, lymphoma, myeloma, neuroblastoma, ovarian, pancreatic, prostate, rhabdoid_tumor,
  sarcoma, skin, testis, thyroid, uterine) — priors would not be grounded
- The analysis needs protein abundance, modification status, or subcellular localization
  — Protrec2 priors operate on a binary presence/absence basis only
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3557
- http://edamontology.org/topic_0128
tools:
- name: Protrec2
  role: Provides the built-in updated_tissue_complex_*.csv tissue annotations and
    GO-based coherence scoring that generate lung complex priors
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Loads and aggregates the tissue complex CSVs with pandas to compute per-complex
    prior probabilities
- name: pip
  role: Installs Protrec2 dependencies via requirements.txt before annotation files
    can be processed
merged_aliases:
- tissue-specific-protein-complex-annotation
merged_alias_records:
- alias: tissue-specific-protein-complex-annotation
  slug: tissue-specific-protein-complex-annotation
  jaccard_score: 0.6667
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:941dbb63dda6041ea29ff42eb686fd938eab0a5f05758deb3759b2d17cc879f6
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
  - pip
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3557
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# lung-tissue-specific-complex-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns tissue-specific confidence probabilities to protein complexes using Protrec2's built-in updated_tissue_complex_*.csv annotations for lung, combining explicit tissue relevance scores (1/0.5/0) with GO shared-term coherence fallback to form complex-level priors used in Bayesian missing-protein recovery.

## When to use

Use when a proteomics recovery task needs tissue-aware complex priors — e.g., running Protrec2 with --tissue lung on an expression matrix and requiring per-complex confidence values across 27 tissue types.

## When NOT to use

- Input is an expression matrix you want scored directly — this skill produces complex-level priors, not per-sample protein probabilities (run main.py recovery for that)
- Studying a tissue outside the 27 supported types (bile, bladder, bone, brain, breast, cervical, colorectal, esophageal, gallbladder, gastric, head_and_neck, kidney, leukemia, liver, lung, lymphoma, myeloma, neuroblastoma, ovarian, pancreatic, prostate, rhabdoid_tumor, sarcoma, skin, testis, thyroid, uterine) — priors would not be grounded
- The analysis needs protein abundance, modification status, or subcellular localization — Protrec2 priors operate on a binary presence/absence basis only

## Inputs

- Built-in tissue-specific complex records: data/updated_tissue_complex_*.csv (e.g., updated_tissue_complex_lung.csv)
- Gene Ontology associations for functional coherence: data/GO.tsv
- Protein complex list (CORUM size ≥5, Complex Portal, Reactome, STRING-expanded score >780; 3,885 complexes in data/complex_list.csv)

## Outputs

- Per-complex, per-tissue prior probability table (P_complex_initial) covering all 27 tissue types including lung
- Lung-specific complex confidence values consumed by the Bayesian protein recovery step of Protrec2

## How to apply

Load the built-in tissue-specific complex records from updated_tissue_complex_lung.csv (part of data/updated_tissue_complex_*.csv) with Python/pandas. For each complex record, assign a lung relevance score of 1.0 when annotations explicitly support the tissue, 0.5 when evidence is ambiguous or unspecified, and 0.0 when explicitly non-relevant; for complexes lacking direct annotation, compute a GO shared-term coherence score (from data/GO.tsv) as the fallback relevance estimate. Average the direct annotation and GO coherence values per complex to produce the initial complex probability P_complex_initial(c), which feeds Protrec2's Bayesian inference alongside prior detection probability and cocomplex structural support. These priors are what allow Protrec2 to account for protein expression variation across tissues, a limitation of the original PROTREC that relied only on CORUM's 694 complexes of size ≥5.

## Related tools

- **Protrec2** (Provides the built-in updated_tissue_complex_*.csv tissue annotations and GO-based coherence scoring that generate lung complex priors) — https://github.com/miaomiao6606/Protrec2
- **Python** (Loads and aggregates the tissue complex CSVs with pandas to compute per-complex prior probabilities)
- **pip** (Installs Protrec2 dependencies via requirements.txt before annotation files can be processed)

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- The output table covers all 27 tissue types from the tissue-name sheet and contains a finite prior probability for every complex-tissue pair
- Every per-tissue relevance score is one of {1.0, 0.5, 0.0} for directly annotated complexes, with GO coherence used only as fallback
- Using --tissue lung selects the lung column of updated_tissue_complex_lung.csv; switching tissue changes the prior values monotonically with annotation support
- The resulting priors, combined with detection probability and cocomplex support in Bayesian inference, yield >85% CPTAC-supported recovered proteins in lung tumor-normal validation

## Limitations

- Tissue-specific annotations were derived from population-level data and may not reflect interpatient heterogeneity
- GO coherence is only a fallback relevance estimate for complexes lacking direct tissue annotation, so priors for those complexes are weaker
- The underlying complex repository (3,885 complexes) remains incomplete for novel or condition-specific complexes

## Evidence

- [other] Tissue annotation averaging with GO fallback: [evidence span withheld — anchor status: not_found]
