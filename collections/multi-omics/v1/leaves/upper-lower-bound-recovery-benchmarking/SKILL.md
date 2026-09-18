---
name: upper-lower-bound-recovery-benchmarking
description: Use when evaluating how many recovered/missing proteins a prediction
  method truly validates, by masking a DDA-PASEF or Astral-derived expression matrix
  under upper- and lower-bound scenarios and checking recovery against external reference
  sets.
when_to_use_negative:
- When the goal is abundance quantification or differential expression rather than
  binary presence/absence recovery validation — Protrec2-style benchmarking operates
  on a binary presence/absence basis
- When no external reference proteomics dataset (e.g. diaPASEF, Astral, CPTAC) exists
  to score recovered proteins, since reported recovery rates are lower bounds constrained
  by reference coverage
- When input is raw MS spectra rather than a protein-level expression matrix; the
  benchmark expects an already-identified protein list per sample
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Protrec2
  role: 'Method under test: Bayesian tissue-specific protein recovery run on masked
    expression matrices under both benchmark scenarios'
  repo: https://github.com/miaomiao6606/Protrec2
- name: PROTREC
  role: Baseline recovery method run on the same masked inputs for head-to-head recovery-rate
    comparison
- name: Python
  role: Runs main.py pipelines and evaluation scripts for probability cutoffs and
    reference validation
- name: pip
  role: Installs Protrec2 dependencies from requirements.txt
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:fb83ca4a10349fc7dfea23ef678548d7f557f9ad3bc6d3bbe05cc1b977dfe0e6
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# upper-lower-bound-recovery-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark missing-protein recovery methods by masking observed proteins under 'upper-bound' (permissive) and 'lower-bound' (constrained) scenarios, then scoring predicted proteins against external proteomics references (e.g. diaPASEF, Astral) at probability cutoffs from 0.50 to 0.95.

## When to use

Use when evaluating how many recovered/missing proteins a prediction method truly validates, by masking a DDA-PASEF or Astral-derived expression matrix under upper- and lower-bound scenarios and checking recovery against external reference sets.

## When NOT to use

- When the goal is abundance quantification or differential expression rather than binary presence/absence recovery validation — Protrec2-style benchmarking operates on a binary presence/absence basis
- When no external reference proteomics dataset (e.g. diaPASEF, Astral, CPTAC) exists to score recovered proteins, since reported recovery rates are lower bounds constrained by reference coverage
- When input is raw MS spectra rather than a protein-level expression matrix; the benchmark expects an already-identified protein list per sample

## Inputs

- Protein-by-sample expression matrix CSV (rows: UniProt IDs or gene symbols; columns: samples; numeric non-negative values)
- Masking scenario definition (upper-bound and lower-bound observed-protein lists)
- External validation reference sets: diaPASEF multi-omics/meta-analysis reference and Astral HeLa reference (8796 proteins)

## Outputs

- Per-sample recovery result CSVs (Protein,Probability columns) for each method and scenario
- Recovery rates at probability cutoffs 0.50–0.95 per method, scenario, and reference set
- Validated-protein counts against each external reference (e.g. 453 diaPASEF upper-bound, 450 Astral)

## How to apply

Prepare a protein-by-sample expression matrix (CSV, rows = UniProt IDs or gene symbols, columns = samples, e.g. 3-sample HeLa DDA-PASEF data) and run the recovery method under both masking scenarios: the lower-bound scenario validates predictions from a single observed protein list against an external diaPASEF reference, while the upper-bound scenario is the permissive setting. Run baselines (PROTREC, FCS, HE, GSEA) on the identical masked inputs so recovery rates are directly comparable. Collect per-sample output CSVs in Protein,Probability format and evaluate recovered proteins against probability cutoffs from 0.50 to 0.95. Validate against two references — the diaPASEF multi-omics/meta-analysis reference and the Astral HeLa reference (8796 proteins) — and report mean recovery rates plus validated-protein counts. Expected anchors from the Protrec2 HeLa benchmark: lower-bound recovery 74.6% (diaPASEF) and 69.0% (Astral) vs PROTREC 75.5%, FCS 68.7%, HE 62.6%, GSEA 63.4%; upper-bound 96.5% (diaPASEF) and 96.0% (Astral) with 453 (diaPASEF) and 450 (Astral) validated proteins.

## Related tools

- **Protrec2** (Method under test: Bayesian tissue-specific protein recovery run on masked expression matrices under both benchmark scenarios) — https://github.com/miaomiao6606/Protrec2
- **PROTREC** (Baseline recovery method run on the same masked inputs for head-to-head recovery-rate comparison)
- **Python** (Runs main.py pipelines and evaluation scripts for probability cutoffs and reference validation)
- **pip** (Installs Protrec2 dependencies from requirements.txt)

## Examples

```
python main.py --expression hela_expression.csv --output_dir results/ --tissue cervical --fdr 0.01 --threshold 5
```

## Evaluation signals

- Lower-bound HeLa recovery validated against diaPASEF should approximate 74.6% for Protrec2 (69.0% against Astral), and other methods should fall below ~76% (PROTREC 75.5%, FCS 68.7%, HE 62.6%, GSEA 63.4%)
- Upper-bound recovery should reach 96.5% (diaPASEF) and 96.0% (Astral) on HeLa, with 453 and 450 validated proteins respectively
- Recovery rate curves must improve monotonically as the probability cutoff increases from 0.50 to 0.95
- Output CSVs must parse with exactly Protein,Probability columns and probabilities in [0,1]
- Baselines (PROTREC, FCS, HE, GSEA) must be run on identical masked inputs and identical FDR/threshold settings for comparability

## Limitations

- Reported recovery rates are lower bounds on true performance because validation performance is constrained by proteomics dataset coverage
- Tissue-specific annotations were derived from population-level data and might not fully reflect interpatient heterogeneity
- Protrec2 operates on a binary presence/absence basis and does not incorporate protein abundance, modification status, or subcellular localization

## Evidence

- [readme] Output file schema and probability interpretation: [evidence span withheld — anchor status: external_doc]
