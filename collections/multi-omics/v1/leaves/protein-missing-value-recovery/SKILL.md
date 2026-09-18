---
name: protein-missing-value-recovery
description: Use when a DDA proteomics expression matrix (rows = proteins, columns
  = samples) contains missing proteins and you need probabilistic presence predictions
  validated against an independent reference (e.g. diaPASEF, Astral, CPTAC).
when_to_use_negative:
- You need protein abundance, modification status, or subcellular localization — Protrec2
  operates on a binary presence/absence basis only
- The question concerns novel or condition-specific protein complexes absent from
  the built-in repository (CORUM, Complex Portal, Reactome, STRING), since the complex
  repository remains incomplete for these
- The dataset requires tissue- or patient-specific priors beyond the population-level
  annotations, which may not reflect interpatient heterogeneity
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Protrec2
  role: 'Primary framework: Bayesian inference over tissue-specific complex annotations
    to output per-protein recovery probabilities'
  repo: https://github.com/miaomiao6606/Protrec2
- name: PROTREC
  role: CORUM-based baseline recovery method run on the same expression input for
    benchmarking recovery rates
- name: Python
  role: Runtime for main.py, per-sample CSV handling, and evaluation at probability
    cutoffs 0.50-0.95
- name: pip
  role: Dependency installation via pip install -r requirements.txt
merged_aliases:
- missing-protein-recovery-benchmarking
merged_alias_records:
- alias: missing-protein-recovery-benchmarking
  slug: missing-protein-recovery-benchmarking
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:6c56636e84be2499cf8a21fe51cbe2348ea2362ff632502afc93a10a6cb60158
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.25
n_claims: 12
---

# protein-missing-value-recovery

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Recover unreported but biologically present proteins in mass spectrometry proteomics using Protrec2, which combines tissue-specific protein complex annotations with Bayesian inference over prior detection probability, cocomplex support, and tissue confidence.

## When to use

Use when a DDA proteomics expression matrix (rows = proteins, columns = samples) contains missing proteins and you need probabilistic presence predictions validated against an independent reference (e.g. diaPASEF, Astral, CPTAC).

## When NOT to use

- You need protein abundance, modification status, or subcellular localization — Protrec2 operates on a binary presence/absence basis only
- The question concerns novel or condition-specific protein complexes absent from the built-in repository (CORUM, Complex Portal, Reactome, STRING), since the complex repository remains incomplete for these
- The dataset requires tissue- or patient-specific priors beyond the population-level annotations, which may not reflect interpatient heterogeneity

## Inputs

- Expression matrix CSV (rows: UniProt IDs or gene symbols; columns: samples; non-negative numeric values, e.g. example/example_expression.csv)
- Tissue type for the prior (one of 27 built-in tissues, e.g. lung)
- Tissue-specific complex prior files: data/updated_tissue_complex_*.csv
- Validation reference set for recovery-rate evaluation (e.g. diaPASEF, Astral HeLa 8796 proteins, CPTAC)
- Parameters: --fdr 0.01, --threshold 5, --fnr 0.0 (defaults)

## Outputs

- Per-sample CSV files at {output_dir}/PROTREC2_result_{sample_name}.csv with columns Protein,Probability
- Recovery-rate table under lower-bound and upper-bound scenarios (e.g. recovery_rate_table.csv, ablation_results.csv)
- Validated-protein counts per sample (e.g. 453 diaPASEF upper-bound, 450 Astral in HeLa)

## How to apply

Install the Protrec2 package with 'pip install -r requirements.txt' and provide an expression matrix CSV (rows: UniProt IDs or gene symbols; columns: samples; non-negative numeric values). Run 'python main.py' selecting a tissue-specific prior from the built-in updated_tissue_complex_*.csv files (27 tissues, e.g. --tissue lung), with --fdr 0.01 and --threshold 5 (minimum complex size). The method generates 3,885 protein complexes from CORUM (size >= 5), STRING expansion (score > 780), Complex Portal, and Reactome, then applies Bayesian inference integrating prior detection probability, cocomplex structural support, and tissue-specific complex confidence with logit-space smoothing. Collect per-sample output CSVs (Protein,Probability) and evaluate recovered proteins against a validation set at probability cutoffs from 0.50 to 0.95, computing recovery rates under lower-bound and upper-bound validation scenarios. Judge correctness by reproducing reported benchmarks, e.g. 90.8% lower-bound / 98.4% upper-bound recovery in A549 and 74.6% / 96.5% in HeLa, and monotonic improvement of recovery rate from cutoff 0.50 to 0.95.

## Related tools

- **Protrec2** (Primary framework: Bayesian inference over tissue-specific complex annotations to output per-protein recovery probabilities) — https://github.com/miaomiao6606/Protrec2
- **PROTREC** (CORUM-based baseline recovery method run on the same expression input for benchmarking recovery rates)
- **Python** (Runtime for main.py, per-sample CSV handling, and evaluation at probability cutoffs 0.50-0.95)
- **pip** (Dependency installation via pip install -r requirements.txt)

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- Per-sample output CSVs follow the exact schema 'Protein,Probability' with probabilities in 0-1; >0.9 indicates strong evidence
- A549 benchmark reproduces mean lower-bound recovery ~90.8% (vs PROTREC 81.8%, FCS 82.8%, HE 80.1%, GSEA 73.8%) and upper-bound ~98.4%
- HeLa benchmark reproduces lower-bound recovery 74.6% against diaPASEF (69.0% against Astral) and upper-bound 96.5% (96.0% Astral) with 453 diaPASEF / 450 Astral validated proteins
- Recovery rate shows monotonic improvement from cutoff 0.50 to 0.95 (threshold sensitivity, Supplementary Figs S3-S4)
- Under a unified multi-source complex-set ablation, Protrec2 still reaches 96.0% recovery / 450.3 validated proteins (upper bound, Astral HeLa) while GSEA precision stays near 36.7%

## Limitations

- Protrec2 operates on a binary presence/absence basis and does not incorporate protein abundance, modification status, or subcellular localization
- Tissue-specific annotations are derived from population-level data and may not reflect interpatient heterogeneity
- The protein complex repository is incomplete for novel or condition-specific complexes
- Validation performance is constrained by proteomics dataset coverage; reported recovery rates are lower bounds on true performance
- Large datasets can trigger memory errors; process samples in batches or reduce the complex size threshold (--threshold)

## Evidence

- [readme] README output format: Protein,Probability per sample: [evidence span withheld — anchor status: external_doc]
