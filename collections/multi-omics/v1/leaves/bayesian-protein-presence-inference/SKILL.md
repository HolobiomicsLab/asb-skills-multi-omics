---
name: bayesian-protein-presence-inference
description: 'Use when recovering missing (unreported but likely present) proteins
  from a proteomics expression matrix (rows: UniProt/gene IDs, columns: samples) using
  tissue-specific protein complex priors and Bayesian inference.'
when_to_use_negative:
- The task requires protein abundance, modification status, or subcellular localization
  inference — Protrec2 operates on a binary presence/absence basis only
- The protein complexes of interest are novel or condition-specific and absent from
  the curated repository, or the tissue type is not among the 27 available tissue
  annotations
- The input is not a protein-level expression matrix (e.g. already a peptide-level
  feature table or a validated presence list); Protrec2 expects UniProt-ID/gene-symbol
  rows with non-negative expression values
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0128
tools:
- name: Protrec2
  role: Implements the Bayesian three-evidence-layer inference over prior detection
    probability, cocomplex support, and tissue-specific complex confidence, invoked
    via main.py
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Runtime for main.py and for loading the expression matrix and tissue complex
    priors
- name: pip
  role: Installs Protrec2 dependencies via requirements.txt before running the inference
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:95e86485c61c49800bcf586aa6e0923cd53ee7b16d8202e01d819283b3400161
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
  - PROTREC
  - Protrec2
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# bayesian-protein-presence-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Protrec2's Bayesian protein-presence inference combines three evidence layers — prior detection probability, co-complex structural support, and tissue-specific complex confidence — via logit-space averaging to update the presence probability of unobserved proteins in mass-spectrometry proteomics expression matrices.

## When to use

Use when recovering missing (unreported but likely present) proteins from a proteomics expression matrix (rows: UniProt/gene IDs, columns: samples) using tissue-specific protein complex priors and Bayesian inference.

## When NOT to use

- The task requires protein abundance, modification status, or subcellular localization inference — Protrec2 operates on a binary presence/absence basis only
- The protein complexes of interest are novel or condition-specific and absent from the curated repository, or the tissue type is not among the 27 available tissue annotations
- The input is not a protein-level expression matrix (e.g. already a peptide-level feature table or a validated presence list); Protrec2 expects UniProt-ID/gene-symbol rows with non-negative expression values

## Inputs

- Expression matrix CSV (rows: proteins as UniProt IDs or gene symbols; columns: samples; non-negative numeric values), e.g. example/example_expression.csv
- Tissue-specific complex prior from built-in updated_tissue_complex_*.csv files (27 tissues, e.g. 'lung' for A549)
- Parameters: --fdr 0.01, --threshold 5 (minimum complex size), --fnr 0.0
- GO.tsv gene-ontology annotations used for functional coherence scoring

## Outputs

- One CSV per sample at {output_dir}/PROTREC2_result_{sample_name}.csv with columns 'Protein' and 'Probability' (recovery probabilities in [0,1])
- Recovered protein lists restricted to proteins meeting the 0.95 confidence cutoff (or evaluated across cutoffs 0.50–0.95)

## How to apply

Load the expression matrix CSV (e.g. example/example_expression.csv) and the built-in tissue-specific complex prior (updated_tissue_complex_*.csv; 3,885 complexes integrating CORUM, Complex Portal, Reactome, and STRING-expanded interactions with confidence > 780). Initialize priors: observed proteins get 1 - FDR (FDR = 0.01 via --fdr 0.01) and unobserved proteins get 0.1. For each unobserved protein, compute the co-complex conditional probability from observed proteins co-occurring in tissue-annotated complexes passing the size threshold (--threshold 5), then combine prior, co-complex support, and complex confidence with equal weights (alpha = beta = gamma = 1/3) by averaging in logit space and mapping back through the sigmoid; logit-space smoothing prevents extreme values from dominating the update. Apply z-score then min-max normalization to the recovered probabilities and retain proteins at or above the 0.95 confidence cutoff, also evaluating monotonic recovery across cutoffs from 0.50 to 0.95. Select the tissue matching the sample source (e.g. --tissue lung for A549) since tissue relevance scores (1.0 explicit support, 0.5 ambiguous, 0.0 non-relevant) and GO-based functional coherence feed the tissue-specific prior.

## Related tools

- **Protrec2** (Implements the Bayesian three-evidence-layer inference over prior detection probability, cocomplex support, and tissue-specific complex confidence, invoked via main.py) — https://github.com/miaomiao6606/Protrec2
- **Python** (Runtime for main.py and for loading the expression matrix and tissue complex priors)
- **pip** (Installs Protrec2 dependencies via requirements.txt before running the inference)

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- Each output CSV exists, parses with columns 'Protein' and 'Probability', and contains probabilities within [0,1]
- Output CSVs contain only proteins meeting the 0.95 confidence threshold
- Benchmarked recovery-rate reproduction: lower-bound A549 mean recovery 90.8% (vs PROTREC 81.8%, FCS 82.8%, HE 80.1%, GSEA 73.8%) and upper-bound 98.4% with 650 proteins validated
- HeLa lower-bound recovery 74.6% against diaPASEF reference (69.0% against Astral 8796-protein set) and upper-bound 96.5% with 453 validated proteins
- Monotonic improvement of recovery rates as the probability cutoff increases from 0.50 to 0.95

## Limitations

- Tissue-specific annotations were derived from population-level data and might not fully reflect interpatient heterogeneity
- Operates on binary presence/absence and does not incorporate protein abundance, modification status, or subcellular localization
- The protein complex repository remains incomplete for novel or condition-specific complexes
- Validation performance is constrained by proteomics dataset coverage; reported recovery rates represent lower bounds on true performance

## Evidence

- [methods] Combines three evidence layers via logit-space averaging with equal weights: [evidence span withheld — anchor status: not_found]
- [discussion] Bayesian inference model over three evidence layers: "Protrec2 employs a Bayesian inference model that integrates three complementary layers of evidence—prior detection probability, cocomplex structural support, and tissue-specific complex"
- [discussion] Logit-space smoothing stabilizes inference: "This probabilistic formulation not only stabilizes inference through logit-space smoothing but also enables nuanced recovery under sparse or noisy data conditions"
- [methods] Initialize priors from observation status with FDR: [evidence span withheld — anchor status: external_doc]
- [readme] Bayesian inference for recovering unreported proteins: [evidence span withheld — anchor status: external_doc]
- [readme] Iterative Bayesian update integrating observation, complex membership, and tissue prior: [evidence span withheld — anchor status: not_found]
