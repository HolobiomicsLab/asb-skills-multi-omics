---
name: tissue-annotation-matrix-construction
description: Decision support for building tissue-specific complex priors from built-in
  updated_tissue_complex_*.csv records when direct tissue annotations are missing
  or ambiguous and GO coherence must be used as a fallback before Bayesian recovery.
when_to_use_negative:
- Input is a protein expression matrix for running the full Protrec2 recovery pipeline
  — this skill only builds the tissue prior, not the Bayesian protein recovery itself
- The tissue of interest is not among the 27 supported types (e.g. plant or microbial
  samples) — no annotation records or GO fallback coverage exist for it
- Complexes needed are novel or condition-specific assemblies absent from the curated
  complex repository, which the authors note 'remains incomplete for novel or condition-specific
  complexes'
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0089
tools:
- name: Protrec2
  role: Framework whose tissue-annotation stage consumes the constructed per-complex
    per-tissue probability matrix as the tissue-specific complex confidence layer
    of the Bayesian inference model
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Used with pandas to load updated_tissue_complex_*.csv, compute per-tissue
    relevance scores and GO shared-term coherence fallbacks, and average them into
    the prior table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:26872435c0317dffc6e9b7ff5d365f66cb8ac04ce78ea4c4cf789d8c99853f52
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# tissue-annotation-matrix-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a tissue-specific complex probability matrix for Protrec2 by scoring each protein complex per tissue (1 = explicit support, 0.5 = ambiguous, 0 = explicitly non-relevant across 27 tissues) and using GO shared-term coherence as a fallback for complexes lacking direct annotation, yielding per-complex per-tissue prior probabilities P_complex_initial(c).

## When to use

Decision support for building tissue-specific complex priors from built-in updated_tissue_complex_*.csv records when direct tissue annotations are missing or ambiguous and GO coherence must be used as a fallback before Bayesian recovery.

## When NOT to use

- Input is a protein expression matrix for running the full Protrec2 recovery pipeline — this skill only builds the tissue prior, not the Bayesian protein recovery itself
- The tissue of interest is not among the 27 supported types (e.g. plant or microbial samples) — no annotation records or GO fallback coverage exist for it
- Complexes needed are novel or condition-specific assemblies absent from the curated complex repository, which the authors note 'remains incomplete for novel or condition-specific complexes'

## Inputs

- Built-in tissue-specific complex records: data/updated_tissue_complex_*.csv
- Gene Ontology term associations: data/GO.tsv
- Protein complex list integrating CORUM, Complex Portal, Reactome, and STRING-expanded complexes (data/complex_list.csv, human_complexes_2022.txt)

## Outputs

- Tissue-specific complex prior probability table: one finite probability per complex per tissue (P_complex_initial(c)) for all 27 tissue types

## How to apply

Load the built-in tissue-specific complex records (data/updated_tissue_complex_*.csv) with Python/pandas and score each complex record per tissue: 1.0 for explicit tissue support, 0.0 for explicitly non-relevant, and 0.5 for ambiguous or unspecified evidence, across the 27 supported tissues (lung, breast, brain, bile, bladder, bone, cervical, colorectal, etc.). For complexes lacking direct tissue annotation, compute a Gene Ontology shared-term coherence score using data/GO.tsv as the fallback tissue relevance estimate, as described in the README's tissue-annotation step. Average the direct annotations and the GO coherence fallback per complex per tissue to produce P_complex_initial(c), then write the result as a CSV table with one finite probability per complex-tissue pair. Rationale: PROTREC's original dependence on CORUM alone (only 694 human complexes of size ≥5) limited coverage, so Protrec2 expands to 3,885 complexes (CORUM, Complex Portal, Reactome, STRING-expanded with confidence > 780) and anchors Bayesian inference with tissue-specific complex confidence as one of its three evidence layers.

## Related tools

- **Protrec2** (Framework whose tissue-annotation stage consumes the constructed per-complex per-tissue probability matrix as the tissue-specific complex confidence layer of the Bayesian inference model) — https://github.com/miaomiao6606/Protrec2
- **Python** (Used with pandas to load updated_tissue_complex_*.csv, compute per-tissue relevance scores and GO shared-term coherence fallbacks, and average them into the prior table)

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- Output CSV table covers all 27 tissue types listed in the README's available-tissue list (bile through uterine) with one row per complex and tissue
- Every complex-tissue pair in the table contains a finite probability value in [0, 1]
- Directly annotated complexes receive scores of exactly 1.0, 0.5, or 0.0 per the README's scoring rule; only unannotated complexes rely on the GO coherence fallback
- Complex records derive from the 3,885 unique complexes in the built-in repository (CORUM size ≥ 5, STRING confidence > 780, >80% valid UniProt IDs)
- Downstream sanity check: Protrec2 run with these priors on lung tissue recovers proteins with monotonic behavior across probability cutoffs 0.50–0.95

## Limitations

- Tissue-specific annotations were derived from population-level data and may not reflect interpatient heterogeneity
- The protein complex repository remains incomplete for novel or condition-specific complexes
- GO-based coherence is only a fallback estimate; complexes with genuinely ambiguous or partial tissue evidence receive the intermediate 0.5 score, which may dilute tissue specificity

## Evidence
