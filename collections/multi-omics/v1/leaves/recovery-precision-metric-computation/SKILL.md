---
name: recovery-precision-metric-computation
description: Use when scoring predicted missing-protein recoveries against a reference
  proteome — e.g., comparing Protrec2/PROTREC/GSEA outputs by recovery rate and validated-protein
  counts across probability cutoffs and upper-/lower-bound scenarios.
when_to_use_negative:
- The method output is a raw expression matrix rather than probability-ranked recovered-protein
  lists, so no recovery-rate/validated-protein scoring is possible
- The task is to predict or recover missing proteins itself (run Protrec2's Bayesian
  inference) rather than to evaluate recovery precision against a reference
- No reference proteome (e.g., Astral HeLa) or CPTAC-style validation set is available
  to define upper-/lower-bound scenarios
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Protrec2
  role: Produces per-sample probability-ranked recovery outputs whose precision is
    computed; benchmarked at 96.0%/450.3 (upper) and 69.0%/41.7 (lower) on the Astral
    HeLa ablation
  repo: https://github.com/miaomiao6606/Protrec2
- name: PROTREC
  role: Baseline missing-protein recovery method whose outputs are scored under the
    same upper-/lower-bound evaluation for comparison
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Runtime for main.py and for computing per-cutoff recovery metrics into ablation_results.csv
  repo: https://github.com/miaomiao6606/Protrec2
- name: pip
  role: Installs package dependencies from requirements.txt before running the evaluation
  repo: https://github.com/miaomiao6606/Protrec2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:7fed3cf850f855848ac8f73a1baa13634f743e99011795a6201c8a113d62427f
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# recovery-precision-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computation of recovery-precision metrics for missing-protein recovery methods: per-sample probability-ranked recoveries are scored against a reference proteome (e.g., Astral HeLa) to yield recovery rate and number of validated proteins under upper-bound and lower-bound scenarios, scanning probability cutoffs from 0.50 to 0.95.

## When to use

Use when scoring predicted missing-protein recoveries against a reference proteome — e.g., comparing Protrec2/PROTREC/GSEA outputs by recovery rate and validated-protein counts across probability cutoffs and upper-/lower-bound scenarios.

## When NOT to use

- The method output is a raw expression matrix rather than probability-ranked recovered-protein lists, so no recovery-rate/validated-protein scoring is possible
- The task is to predict or recover missing proteins itself (run Protrec2's Bayesian inference) rather than to evaluate recovery precision against a reference
- No reference proteome (e.g., Astral HeLa) or CPTAC-style validation set is available to define upper-/lower-bound scenarios

## Inputs

- Per-sample method output CSVs with recovered proteins and recovery probabilities (Protein,Probability columns)
- Astral HeLa reference proteome (validated protein presence per sample)
- Expanded multi-source complex prior (updated_tissue_complex_*.csv) shared across all methods
- HeLa expression matrix (rows = proteins by UniProt ID or symbol, columns = samples)

## Outputs

- Per-cutoff, per-scenario recovery rate for each method
- Number of validated proteins per sample for each method
- Aggregated comparison table (e.g., ablation_results.csv) with upper-bound and lower-bound metrics

## How to apply

Collect each method's per-sample output CSVs (e.g., PROTREC2_result_{sample}.csv with Protein/Probability columns) and rank recovered proteins by probability. Evaluate each ranked list against the Astral HeLa reference proteome under both the upper-bound (permissive) and lower-bound (constrained) scenarios, sweeping probability cutoff thresholds from 0.50 to 0.95 and computing, at each cutoff, the recovery rate (fraction of predictions validated as present in the reference) and the number of validated proteins per sample. Run Protrec2 with --fdr 0.01 and --threshold 5 against the shared expanded multi-source complex prior (updated_tissue_complex_*.csv) so all methods are scored on identical prior knowledge, isolating the Bayesian inference mechanism as the differentiator. Aggregate per-method metrics (e.g., into ablation_results.csv) and expect Protrec2 to reach ~96.0% recovery with 450.3 validated proteins (upper bound) and ~69.0% with 41.7 validated (lower bound) on HeLa, while baselines like GSEA drop to ~36.7% (upper) / 38.5% (lower) validation precision. Confirm monotonic improvement of metrics from cutoff 0.50 to 0.95 as a sanity check on the probabilistic ranking.

## Related tools

- **Protrec2** (Produces per-sample probability-ranked recovery outputs whose precision is computed; benchmarked at 96.0%/450.3 (upper) and 69.0%/41.7 (lower) on the Astral HeLa ablation) — https://github.com/miaomiao6606/Protrec2
- **PROTREC** (Baseline missing-protein recovery method whose outputs are scored under the same upper-/lower-bound evaluation for comparison) — https://github.com/miaomiao6606/Protrec2
- **Python** (Runtime for main.py and for computing per-cutoff recovery metrics into ablation_results.csv) — https://github.com/miaomiao6606/Protrec2
- **pip** (Installs package dependencies from requirements.txt before running the evaluation) — https://github.com/miaomiao6606/Protrec2

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- Protrec2's ablation row reproduces 96.0% recovery / 450.3 validated proteins (upper bound) and 69.0% / 41.7 (lower bound) on Astral HeLa
- GSEA validation precision falls near 36.7% (upper) and 38.5% (lower), sharply below Protrec2
- Metrics improve monotonically as the probability cutoff increases from 0.50 to 0.95
- Output CSVs conform to the Protein/Probability schema and probabilities fall within [0,1]
- All methods were scored against the same expanded multi-source complex set, so metric differences reflect the inference mechanism rather than prior access

## Limitations

- Validation performance is constrained by proteomics dataset coverage; reported recovery rates represent lower bounds on true performance
- Protrec2's binary presence/absence basis means validated-protein counts ignore abundance, modification status, and subcellular localization
- Metrics are sensitive to the reference proteome definition — upper-bound and lower-bound scenarios give substantially different absolute numbers (e.g., 96.0% vs 69.0%)

## Evidence

- [readme] README performance reference figures for HeLa: [evidence span withheld — anchor status: external_doc]
