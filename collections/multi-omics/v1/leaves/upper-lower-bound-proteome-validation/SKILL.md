---
name: upper-lower-bound-proteome-validation
description: Use when evaluating a proteomic missing-protein recovery method against
  a reference proteome and you need constrained (lower-bound) vs permissive (upper-bound)
  validation of per-sample probability-ranked predictions.
when_to_use_negative:
- You only have raw mass-spectrometry spectra or peptide-level IDs — this skill requires
  a protein-level expression matrix and ranked protein recovery probabilities.
- The goal is protein abundance quantification or PTM/localization analysis — Protrec2-based
  recovery operates on a binary presence/absence basis and validation here only checks
  identity presence.
- There is no reference proteome to validate against — neither the upper-bound nor
  lower-bound scenario can be computed without a matched benchmark.
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Protrec2
  role: Primary recovery method being benchmarked; produces per-sample probability-ranked
    recovered proteins (Protrec2/main.py with --fdr 0.01, --threshold 5)
  repo: https://github.com/miaomiao6606/Protrec2
- name: PROTREC
  role: Baseline network-based recovery method (CORUM-dependent) evaluated on the
    same expression matrix and complex set for comparison
- name: GSEA
  role: Baseline enrichment method scored under both validation scenarios to contrast
    with Protrec2's precision
- name: Python
  role: Runtime for running main.py, the baselines, and the cutoff-scan evaluation
    scripts
  repo: https://github.com/miaomiao6606/Protrec2
- name: pip
  role: Dependency installation for the benchmark environment (pip install -r requirements.txt)
  repo: https://github.com/miaomiao6606/Protrec2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:7578aaa42d75f1bde942e98765f3d35e77d6d4acbf13a9f5474578a997eb9825
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# upper-lower-bound-proteome-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking scheme for missing-protein recovery methods (Protrec2, PROTREC, FCS, HE, GSEA) that scores per-sample probability-ranked recovery under 'upper-bound' and 'lower-bound' scenarios on HeLa and A549 proteomes, scanning probability cutoffs from 0.50 to 0.95 and reporting recovery rate and number of validated proteins.

## When to use

Use when evaluating a proteomic missing-protein recovery method against a reference proteome and you need constrained (lower-bound) vs permissive (upper-bound) validation of per-sample probability-ranked predictions.

## When NOT to use

- You only have raw mass-spectrometry spectra or peptide-level IDs — this skill requires a protein-level expression matrix and ranked protein recovery probabilities.
- The goal is protein abundance quantification or PTM/localization analysis — Protrec2-based recovery operates on a binary presence/absence basis and validation here only checks identity presence.
- There is no reference proteome to validate against — neither the upper-bound nor lower-bound scenario can be computed without a matched benchmark.

## Inputs

- Protein expression matrix (CSV; rows: proteins by UniProt ID or symbol, columns: samples)
- Per-sample ranked recovery predictions with probabilities (e.g. PROTREC2_result_{sample}.csv with Protein,Probability columns)
- Reference proteome for the cell line (e.g. Astral HeLa reference)
- Shared protein complex prior given identically to all methods (e.g. updated_tissue_complex_*.csv, CORUM-derived complex list)

## Outputs

- Per-method, per-cutoff recovery rate and number of validated proteins under upper-bound and lower-bound scenarios
- Aggregated benchmark table (e.g. ablation_results.csv) comparing Protrec2 with PROTREC, FCS, HE, and GSEA
- Monotonic cutoff-response curves from probability cutoff 0.50 to 0.95

## How to apply

Run each recovery method on the same expression matrix (rows = proteins by UniProt ID or symbol, columns = samples), e.g. Protrec2 via main.py with --fdr 0.01 and --threshold 5 so all methods consume identical complex priors (e.g. the expanded multi-source updated_tissue_complex_*.csv in ablation settings). For each sample, sort recovered proteins by their output probability and evaluate them against the reference proteome (Astral HeLa / A549) under two scenarios: an upper-bound permissive setting and a lower-bound constrained setting that excludes ambiguous matches. Scan probability cutoffs from 0.50 to 0.95 and at each cutoff compute (a) recovery rate — the fraction of validated predictions — and (b) the number of validated proteins per sample. Aggregate per-method metrics (e.g. into ablation_results.csv) and inspect monotonic improvement across cutoffs; Protrec2 is expected to hold high precision in both scenarios while baselines such as GSEA drop sharply (36.7% upper / 38.5% lower in the HeLa ablation). The rationale is that upper-bound figures reward coverage while lower-bound figures penalize false-positive recovery, exposing methods that inflate validation only under permissive matching.

## Related tools

- **Protrec2** (Primary recovery method being benchmarked; produces per-sample probability-ranked recovered proteins (Protrec2/main.py with --fdr 0.01, --threshold 5)) — https://github.com/miaomiao6606/Protrec2
- **PROTREC** (Baseline network-based recovery method (CORUM-dependent) evaluated on the same expression matrix and complex set for comparison)
- **GSEA** (Baseline enrichment method scored under both validation scenarios to contrast with Protrec2's precision)
- **Python** (Runtime for running main.py, the baselines, and the cutoff-scan evaluation scripts) — https://github.com/miaomiao6606/Protrec2
- **pip** (Dependency installation for the benchmark environment (pip install -r requirements.txt)) — https://github.com/miaomiao6606/Protrec2

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- Protrec2 on HeLa should reach approximately 96.5% recovery rate with about 453 validated proteins in the upper-bound scenario (98.4% and 650 proteins on A549).
- In the unified-complex HeLa ablation, the Protrec2 row of ablation_results.csv should reproduce 96.0%/450.3 validated proteins (upper bound) and 69.0%/41.7 (lower bound).
- Baseline precision should drop sharply under the same priors: GSEA near 36.7% (upper) and 38.5% (lower) in the HeLa ablation.
- Recovery metrics should improve monotonically as the probability cutoff increases from 0.50 to 0.95 for Protrec2.
- In lower-bound evaluations Protrec2 should validate over 90% of predicted proteins on A549 and about 74.6% on HeLa, while other methods exhibit significant performance drops.

## Limitations

- Protrec2 operates on a binary presence/absence basis and does not incorporate protein abundance, modification status, or subcellular localization.
- Tissue-specific annotations are derived from population-level data and may not reflect interpatient heterogeneity.
- The protein complex repository is incomplete for novel or condition-specific complexes.
- Validation performance is constrained by proteomics dataset coverage, so reported recovery rates represent lower bounds on true performance.

## Evidence

- [readme] **HeLa proteomes**: 96.5% recovery rate (453 proteins validated): [evidence span withheld — anchor status: external_doc]
