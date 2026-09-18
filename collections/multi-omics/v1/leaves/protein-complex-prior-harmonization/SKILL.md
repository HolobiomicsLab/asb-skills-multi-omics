---
name: protein-complex-prior-harmonization
description: Use when benchmarking protein recovery methods on a shared expression
  matrix and you must remove prior-knowledge asymmetry — e.g. testing whether Protrec2's
  advantage derives from its Bayesian inference rather than its expanded 3,885-complex
  multi-source prior.
when_to_use_negative:
- When the goal is to showcase each method's own built-in prior (e.g. PROTREC's CORUM-only
  694 complexes of size ≥5) rather than isolating algorithmic differences — harmonizing
  the prior would destroy that comparison
- When the complex prior is method-specific and cannot be exported (e.g. a baseline
  requires its own internal complex definition), so identical priors cannot be shared
  across methods
- When evaluating tissue specificity itself — the expanded tissue-specific prior is
  the treatment variable and must not be held constant
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0128
tools:
- name: Protrec2
  role: Recovery method run with the expanded multi-source complex prior via main.py
    (--tissue, --fdr 0.01, --threshold 5) to test whether its Bayesian inference drives
    the advantage
  repo: https://github.com/miaomiao6606/Protrec2
- name: PROTREC
  role: Baseline recovery method given the same expanded complex set to remove its
    CORUM-only (694 complexes ≥5 proteins) prior disadvantage
- name: GSEA
  role: Baseline enrichment method run on the identical HeLa expression matrix and
    expanded complex set; its lower precision (36.7% upper, 38.5% lower) quantifies
    the gap
- name: pip
  role: Installs package dependencies via requirements.txt before running the harmonized
    ablation
- name: Python
  role: Runtime for main.py and the evaluation/ablation aggregation scripts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:76c9f3e5b8da884be13a4d7f05d5b74cf9f911f3ddf0fe8adfe414fc54360c99
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
  - GSEA
  - PROTREC
  - Protrec2
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Protein complex prior harmonization for ablation benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Harmonizes the protein complex prior knowledge so that all compared missing-protein recovery methods (Protrec2, PROTREC, GSEA, etc.) consume the identical expanded multi-source complex set, enabling a fair ablation that isolates the algorithmic (Bayesian inference) contribution rather than privileged prior access.

## When to use

Use when benchmarking protein recovery methods on a shared expression matrix and you must remove prior-knowledge asymmetry — e.g. testing whether Protrec2's advantage derives from its Bayesian inference rather than its expanded 3,885-complex multi-source prior.

## When NOT to use

- When the goal is to showcase each method's own built-in prior (e.g. PROTREC's CORUM-only 694 complexes of size ≥5) rather than isolating algorithmic differences — harmonizing the prior would destroy that comparison
- When the complex prior is method-specific and cannot be exported (e.g. a baseline requires its own internal complex definition), so identical priors cannot be shared across methods
- When evaluating tissue specificity itself — the expanded tissue-specific prior is the treatment variable and must not be held constant

## Inputs

- Expression matrix CSV (rows: proteins by UniProt ID or symbol; columns: samples; numeric non-negative values), e.g. the HeLa expression matrix
- Expanded multi-source complex prior CSV (`updated_tissue_complex_*.csv`) shared across all methods, integrating CORUM, Complex Portal, Reactome, and STRING-expanded complexes (score > 780)
- Astral HeLa reference proteome for upper-bound/lower-bound validation
- Baseline method executables/scripts: PROTREC and GSEA

## Outputs

- Per-sample CSV files of recovered protein probabilities (`PROTREC2_result_{sample_name}.csv` with Protein and Probability columns)
- Aggregated per-method ablation metrics table (`ablation_results.csv`) with recovery rate and number of validated proteins per cutoff (0.50–0.95) per scenario (upper/lower bound)

## How to apply

Load the HeLa (or A549) expression matrix with rows as proteins (UniProt ID or symbol) and columns as samples, plus the expanded multi-source complex prior built from `updated_tissue_complex_*.csv`, which integrates CORUM (complexes of size ≥5), Complex Portal, Reactome, and STRING-expanded complexes (confidence > 780) into 3,885 unique complexes. Feed this same comprehensive complex set to every method — Protrec2 via `main.py` with `--tissue` set for the expanded prior, `--fdr 0.01`, `--threshold 5`, and the baselines PROTREC and GSEA on the identical matrix and prior. Evaluate per-sample probability-ranked recovery against the Astral HeLa reference under upper-bound and lower-bound scenarios, scanning probability cutoffs from 0.50 to 0.95 and computing recovery rate and number of validated proteins. The rationale is that when the complex prior is equalized, any residual performance gap (e.g. Protrec2's 96.0% recovery and 450.3 validated proteins vs GSEA's 36.7% in the upper-bound scenario) must be attributable to the inference mechanism, not prior breadth.

## Related tools

- **Protrec2** (Recovery method run with the expanded multi-source complex prior via main.py (--tissue, --fdr 0.01, --threshold 5) to test whether its Bayesian inference drives the advantage) — https://github.com/miaomiao6606/Protrec2
- **PROTREC** (Baseline recovery method given the same expanded complex set to remove its CORUM-only (694 complexes ≥5 proteins) prior disadvantage)
- **GSEA** (Baseline enrichment method run on the identical HeLa expression matrix and expanded complex set; its lower precision (36.7% upper, 38.5% lower) quantifies the gap)
- **pip** (Installs package dependencies via requirements.txt before running the harmonized ablation)
- **Python** (Runtime for main.py and the evaluation/ablation aggregation scripts)

## Examples

```
python main.py --expression hela_expression.csv --output_dir results/ --tissue cervical --fdr 0.01 --threshold 5
```

## Evaluation signals

- The Protrec2 ablation row of ablation_results.csv reproduces 96.0% recovery / 450.3 validated proteins (upper bound) and 69.0% / 41.7 (lower bound) on the Astral HeLa reference
- GSEA precision with the shared expanded prior falls near 36.7% (upper bound) and 38.5% (lower bound), showing a large gap despite identical priors
- All methods consumed the same complex prior — verify identical prior file / complex count (3,885 unique complexes) was loaded for each run
- Recovery metrics show monotonic improvement as probability cutoffs rise from 0.50 to 0.95, matching the reported behavior
- Per-sample output CSVs contain Protein and Probability columns with probabilities in 0–1 and high-confidence entries (>0.9) flagged as strong evidence

## Limitations

- Tissue-specific annotations were derived from population-level data and may not fully reflect interpatient heterogeneity
- Protrec2 operates on a binary presence/absence basis and does not incorporate protein abundance, modification status, or subcellular localization
- The protein complex repository, while comprehensive (3,885 complexes), remains incomplete for novel or condition-specific complexes
- Validation performance is constrained by proteomics dataset coverage; reported recovery rates represent lower bounds on true performance

## Evidence

- [results] Even when all methods were given the same comprehensive multi-source complex set on the Astral HeLa dataset, Protrec2 maintained superior performance with a recovery rate of 96.0%: [evidence span withheld — anchor status: not_found]
- [readme] Total: 3,885 unique protein complexes: [evidence span withheld — anchor status: external_doc]
