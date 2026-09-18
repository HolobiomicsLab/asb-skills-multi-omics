---
name: protein-complex-network-inference
description: 'Use when a proteomics expression matrix (rows: UniProt proteins, columns:
  samples) has persistent missing values and the goal is to predict which absent-but-expected
  proteins are biologically present using complex co-membership evidence.'
when_to_use_negative:
- The task requires protein abundance, modification status, or subcellular localization
  — Protrec2 operates only on a binary presence/absence basis
- The study focuses on novel or condition-specific protein complexes absent from CORUM/Complex
  Portal/Reactome repositories
- Strict reliance on interpatient-specific tissue relevance is needed — tissue annotations
  were derived from population-level data and may not reflect interpatient heterogeneity
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0128
tools:
- name: Protrec2
  role: 'Primary method: Bayesian protein recovery over tissue-specific complex priors,
    run via python main.py'
  repo: https://github.com/miaomiao6606/Protrec2
- name: PROTREC
  role: Baseline recovery method using CORUM-derived complex filtering, used for benchmark
    comparison
- name: Python
  role: Runtime for main.py; loads expression matrix, writes per-sample probability
    CSVs
- name: pip
  role: Installs Protrec2 dependencies from requirements.txt
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:2b6ce47286623421219f316bbffe676107142ff97cf3d55d0cb014afe8e2b23f
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
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.2857142857142857
n_claims: 7
---

# protein-complex-network-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Recover unreported but biologically present ('missing') proteins from DDA-PASEF proteomics expression matrices using Protrec2, which integrates tissue-specific protein complex annotations with Bayesian inference. Outperforms PROTREC, FCS, HE, and GSEA on A549 and HeLa benchmarks.

## When to use

Use when a proteomics expression matrix (rows: UniProt proteins, columns: samples) has persistent missing values and the goal is to predict which absent-but-expected proteins are biologically present using complex co-membership evidence.

## When NOT to use

- The task requires protein abundance, modification status, or subcellular localization — Protrec2 operates only on a binary presence/absence basis
- The study focuses on novel or condition-specific protein complexes absent from CORUM/Complex Portal/Reactome repositories
- Strict reliance on interpatient-specific tissue relevance is needed — tissue annotations were derived from population-level data and may not reflect interpatient heterogeneity

## Inputs

- Expression matrix CSV (rows: UniProt IDs or gene symbols; columns: samples; non-negative numeric values), e.g. example/example_expression.csv
- Tissue selection for the tissue-specific complex prior (27 tissues, e.g. lung)
- Parameters: --fdr (default 0.01), --fnr (default 0.0), --threshold (minimum complex size, default 5)

## Outputs

- Per-sample CSV files `{output_dir}/PROTREC2_result_{sample_name}.csv` with columns Protein,Probability (recovery probability 0-1)
- Recovery-rate tables under lower-bound and upper-bound validation (e.g. Protrec2 90.8%/98.4% in A549)

## How to apply

Install Protrec2 dependencies via `pip install -r requirements.txt`, then run `python main.py` with the expression matrix (`--expression`, rows = proteins, columns = samples), a tissue prior (`--tissue lung` from built-in updated_tissue_complex_*.csv files), `--fdr 0.01`, and a minimum complex size `--threshold 5`. Protrec2 generates a comprehensive complex database (CORUM size ≥5, STRING-expanded with confidence >780, Complex Portal, Reactome; 3,885 complexes total) and assigns tissue relevance scores (1.0 explicit support, 0.5 ambiguous, 0.0 non-relevant) plus GO-based functional coherence. Bayesian inference then iteratively updates each protein's presence likelihood from three evidence layers — prior detection probability, cocomplex structural support, and tissue-specific complex confidence — stabilized by logit-space smoothing. Write per-sample probability CSVs and, for validation, cross-reference recovered proteins against an external validation set (e.g. diaPASEF for A549, CPTAC for lung tumor-normal pairs) under lower- and upper-bound modes across probability cutoffs from 0.50 to 0.95.

## Related tools

- **Protrec2** (Primary method: Bayesian protein recovery over tissue-specific complex priors, run via python main.py) — https://github.com/miaomiao6606/Protrec2
- **PROTREC** (Baseline recovery method using CORUM-derived complex filtering, used for benchmark comparison)
- **Python** (Runtime for main.py; loads expression matrix, writes per-sample probability CSVs)
- **pip** (Installs Protrec2 dependencies from requirements.txt)

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- Per-sample output CSVs contain exactly two columns, Protein and Probability, with probabilities in [0,1] and high-confidence recoveries >0.9
- Lower-bound A549 validation recovery rate exceeds 90% (reported 90.8%) and upper-bound reaches 98.4%, matching the published benchmark values
- Protrec2 outperforms the baselines PROTREC (81.8%), FCS (82.8%), HE (80.1%), and GSEA (73.8%) at the evaluated probability cutoffs
- Monotonic improvement in validation performance is observed as the probability cutoff increases from 0.50 to 0.95
- For lung tumor-normal applications, over 85% of predicted proteins are supported by CPTAC

## Limitations

- Tissue-specific annotations were derived from population-level data and might not fully reflect interpatient heterogeneity
- Protrec2 operates on a binary presence/absence basis and does not incorporate protein abundance, modification status, or subcellular localization
- The protein complex repository remains incomplete for novel or condition-specific complexes
- Validation performance is constrained by proteomics dataset coverage; reported recovery rates represent lower bounds on true performance
- Very large expression datasets may cause memory errors — process samples in batches or reduce the complex size threshold

## Evidence
