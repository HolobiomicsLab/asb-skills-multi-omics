---
name: proteomics-recovery-rate-benchmarking
description: 'Use when you have a DDA/DIA proteomics expression matrix (rows: UniProt
  proteins, columns: samples) with missing proteins and an external validation protein
  set, and need to quantify how many unreported-but-present proteins each recovery
  method correctly predicts.'
when_to_use_negative:
- Your data contains no external validation set — recovery rates cannot be computed
  and the Bayesian priors alone are unvalidated
- You need protein abundance quantification, modification status, or subcellular localization
  — Protrec2 operates only on a binary presence/absence basis
- Protein IDs are not UniProt format or expression values are negative — the pipeline
  requires UniProt IDs and non-negative expression levels
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Protrec2
  role: 'Primary method: tissue-specific Bayesian inference framework that recovers
    unreported but biologically present proteins and outputs per-sample recovery probabilities'
  repo: https://github.com/miaomiao6606/Protrec2
- name: PROTREC
  role: 'Baseline method: CORUM-derived complex filtering for missing protein prediction,
    benchmarked against Protrec2'
- name: Python
  role: Execution environment used to run main.py with expression, tissue, FDR, and
    threshold parameters
- name: pip
  role: Dependency installation from requirements.txt
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:eee4ed95a62bb370044564cbdd9d887a8ff12ee6c43c5d3e4be6059ad400f598
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# proteomics-recovery-rate-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking workflow that applies Bayesian network-based missing protein recovery (Protrec2) and baselines (PROTREC, FCS, HE, GSEA) to proteomics expression matrices, then cross-references recovered proteins against external validation sets under lower- and upper-bound scenarios to compute recovery rates.

## When to use

Use when you have a DDA/DIA proteomics expression matrix (rows: UniProt proteins, columns: samples) with missing proteins and an external validation protein set, and need to quantify how many unreported-but-present proteins each recovery method correctly predicts.

## When NOT to use

- Your data contains no external validation set — recovery rates cannot be computed and the Bayesian priors alone are unvalidated
- You need protein abundance quantification, modification status, or subcellular localization — Protrec2 operates only on a binary presence/absence basis
- Protein IDs are not UniProt format or expression values are negative — the pipeline requires UniProt IDs and non-negative expression levels

## Inputs

- Expression matrix CSV (rows: proteins with UniProt IDs or gene symbols; columns: samples; numeric non-negative values), e.g. example/example_expression.csv
- Tissue-specific complex prior from built-in updated_tissue_complex_*.csv files (27 tissues, e.g. lung)
- External validation protein set (e.g. diaPASEF measurements for A549 benchmarking)
- Built-in protein complex databases (CORUM, Complex Portal, Reactome, STRING-expanded complexes; 3,885 complexes total)

## Outputs

- Per-sample recovery CSVs ({output_dir}/PROTREC2_result_{sample_name}.csv) with Protein and Probability (0-1) columns
- Recovery-rate table (recovery_rate_table.csv) reporting lower-bound and upper-bound recovery rates per method (e.g. Protrec2 90.8%/98.4% in A549)

## How to apply

Install Protrec2 via 'pip install -r requirements.txt', then load the expression matrix (rows: proteins as UniProt IDs or gene symbols, columns: samples, e.g. the 24-sample A549 DDA-PASEF benchmark) and select the tissue-specific complex prior from the built-in updated_tissue_complex_*.csv files. Run 'python main.py' with --tissue lung, --fdr 0.01, and --threshold 5, which applies Bayesian inference over three evidence layers — prior detection probability, cocomplex structural support, and tissue-specific complex confidence — and writes per-sample CSVs with (Protein, Probability) columns. Run the baselines (PROTREC with CORUM-derived complex filtering, FCS, HE, GSEA) on the same expression matrix. Cross-reference each method's recovered proteins at probability cutoffs from 0.50 to 0.95 against the external validation set (e.g. diaPASEF for A549) under both lower-bound and upper-bound validation modes. Compile the recovery-rate table and verify it reproduces published values, e.g. Protrec2 90.8%/98.4% in A549 versus PROTREC 81.8%/99.7%, FCS 82.8%/87.4%, HE 80.1%/86.9%, GSEA 73.8%/74.6%.

## Related tools

- **Protrec2** (Primary method: tissue-specific Bayesian inference framework that recovers unreported but biologically present proteins and outputs per-sample recovery probabilities) — https://github.com/miaomiao6606/Protrec2
- **PROTREC** (Baseline method: CORUM-derived complex filtering for missing protein prediction, benchmarked against Protrec2)
- **Python** (Execution environment used to run main.py with expression, tissue, FDR, and threshold parameters)
- **pip** (Dependency installation from requirements.txt)

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- Lower-bound recovery rate for Protrec2 on the A549 benchmark should exceed 90% (reported 90.8%) versus PROTREC 81.8%, FCS 82.8%, HE 80.1%, GSEA 73.8%
- Upper-bound recovery rate should reach 98.4% in A549 (650 proteins validated) and 96.5% in HeLa (453 proteins validated)
- Recovery rates should improve monotonically as probability cutoff increases from 0.50 to 0.95
- Per-sample output CSVs must contain exactly the columns Protein and Probability with values in 0-1
- For lung tumor-normal pairs, over 85% of predicted proteins should be supported by CPTAC validation

## Limitations

- Protrec2 operates on a binary presence/absence basis and does not yet incorporate protein abundance, modification status, or subcellular localization
- Tissue-specific annotations were derived from population-level data and may not reflect interpatient heterogeneity
- The protein complex repository is incomplete for novel or condition-specific complexes
- Validation performance is constrained by proteomics dataset coverage; reported recovery rates represent lower bounds on true performance
- Large datasets can cause memory errors — process samples in batches or reduce the complex size threshold

## Evidence

- [readme] input format: [evidence span withheld — anchor status: external_doc]
- [readme] baseline comparison: [evidence span withheld — anchor status: external_doc]
