---
name: baseline-method-comparison-proteomics
description: 'Use when a recovery method''s advantage must be isolated from prior-knowledge
  effects: all methods share the same expanded multi-source complex set (updated_tissue_complex_.csv)
  on HeLa or A549 expression data before validation against the Astral reference.'
when_to_use_negative:
- A single method's recovery output is needed for downstream use, not a controlled
  head-to-head comparison under equal priors.
- No validated reference set (e.g., Astral HeLa) exists for the dataset, so recovery
  rate and validated-protein counts cannot be computed.
- Input is raw mass-spectrometry data rather than the required CSV expression matrix
  with proteins as rows and samples as columns.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Protrec2
  role: 'Method under test: run via main.py on the HeLa expression matrix with the
    expanded tissue prior (--fdr 0.01, --threshold 5) to produce probability-ranked
    recoveries'
  repo: https://github.com/miaomiao6606/Protrec2
- name: PROTREC
  role: Baseline network-based recovery method run on the identical expression matrix
    and expanded complex set for the equal-prior ablation
- name: GSEA
  role: Baseline gene-set enrichment method evaluated on identical inputs; its validation
    precision (~36.7% upper / ~38.5% lower) quantifies the baseline gap
- name: FCS
  role: Functional Class Scoring baseline listed in the README performance comparison
- name: HE
  role: Hypergeometric Enrichment baseline listed in the README performance comparison
- name: Python
  role: Runtime for main.py and the evaluation scripts that scan cutoffs 0.50–0.95
    and assemble ablation_results.csv
- name: pip
  role: Installs package dependencies from requirements.txt before running any method
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:55d647eb5303669a49691202a5e884317e1b9160254beb7a1e0dff8b6b19e8d1
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
  - FCS
  - GSEA
  - HE
  - PROTREC
  - Protrec2
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# baseline-method-comparison-proteomics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks baseline missing-protein recovery methods (PROTREC, GSEA, plus FCS/HE from the README) against Protrec2 under an identical expanded multi-source complex prior, so residual performance differences are attributable to the Bayesian inference mechanism rather than privileged access to the complex set. Validation uses upper- and lower-bound scenarios against the Astral HeLa reference while scanning probability cutoffs 0.50–0.95.

## When to use

Use when a recovery method's advantage must be isolated from prior-knowledge effects: all methods share the same expanded multi-source complex set (updated_tissue_complex_*.csv) on HeLa or A549 expression data before validation against the Astral reference.

## When NOT to use

- A single method's recovery output is needed for downstream use, not a controlled head-to-head comparison under equal priors.
- No validated reference set (e.g., Astral HeLa) exists for the dataset, so recovery rate and validated-protein counts cannot be computed.
- Input is raw mass-spectrometry data rather than the required CSV expression matrix with proteins as rows and samples as columns.

## Inputs

- HeLa (or A549) expression matrix CSV: rows = proteins (UniProt ID or gene symbol), columns = samples, non-negative numeric values
- Expanded multi-source complex prior: updated_tissue_complex_*.csv (CORUM, Complex Portal, Reactome, STRING-expanded; 3,885 unique complexes) shared by all methods
- Astral HeLa reference protein set for upper- and lower-bound validation
- requirements.txt for environment setup

## Outputs

- ablation_results.csv with per-method recovery rate and validated-protein counts at each probability cutoff (0.50–0.95)
- Per-sample probability-ranked recovery CSVs (e.g., PROTREC2_result_{sample_name}.csv with Protein and Probability columns) for each method

## How to apply

Install dependencies with pip install -r requirements.txt, then load the HeLa expression matrix (rows = proteins by UniProt ID or symbol, columns = samples) together with the expanded multi-source complex prior updated_tissue_complex_*.csv so every method sees identical prior knowledge. Run Protrec2 through main.py with --tissue set for the expanded prior and default --fdr 0.01 and --threshold 5, yielding per-sample PROTREC2_result_{sample}.csv files of probability-ranked recoveries. Run the baselines PROTREC and GSEA (FCS and HE are additional README baselines) on the same expression matrix and the same expanded complex set. Evaluate each method's per-sample probability-ranked recovery against the Astral HeLa reference under upper-bound and lower-bound scenarios, scanning probability cutoffs from 0.50 to 0.95 and computing recovery rate and number of validated proteins per sample. Aggregate per-method metrics into ablation_results.csv; in the unified-prior ablation Protrec2 still reaches 96.0% recovery with 450.3 validated proteins (upper bound) and 69.0% with 41.7 (lower bound), while GSEA validation precision falls near 36.7% (upper) and 38.5% (lower). The equal-prior design is the key decision point: any remaining Protrec2 advantage reflects its Bayesian integration of prior detection probability, cocomplex structural support, and tissue-specific complex confidence.

## Related tools

- **Protrec2** (Method under test: run via main.py on the HeLa expression matrix with the expanded tissue prior (--fdr 0.01, --threshold 5) to produce probability-ranked recoveries) — https://github.com/miaomiao6606/Protrec2
- **PROTREC** (Baseline network-based recovery method run on the identical expression matrix and expanded complex set for the equal-prior ablation)
- **GSEA** (Baseline gene-set enrichment method evaluated on identical inputs; its validation precision (~36.7% upper / ~38.5% lower) quantifies the baseline gap)
- **FCS** (Functional Class Scoring baseline listed in the README performance comparison)
- **HE** (Hypergeometric Enrichment baseline listed in the README performance comparison)
- **Python** (Runtime for main.py and the evaluation scripts that scan cutoffs 0.50–0.95 and assemble ablation_results.csv)
- **pip** (Installs package dependencies from requirements.txt before running any method)

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- Protrec2 ablation row in ablation_results.csv reproduces 96.0% recovery / 450.3 validated proteins (upper bound) and 69.0% / 41.7 (lower bound).
- GSEA validation precision falls near 36.7% (upper) and 38.5% (lower), sharply below Protrec2 under the identical expanded prior.
- All methods consumed the same updated_tissue_complex_*.csv prior — verify no method-specific complex set was used, which would break the ablation logic.
- Per-sample outputs exist as PROTREC2_result_{sample_name}.csv with Protein and Probability columns; scanning cutoffs 0.50 to 0.95 shows monotonic improvement.
- Recovery rate and validated-protein counts are computed against the Astral HeLa reference under both upper-bound and lower-bound scenarios, not just one.

## Limitations

- Validation performance is constrained by proteomics dataset coverage; reported recovery rates represent lower bounds on true performance.
- Protrec2 operates on a binary presence/absence basis and does not incorporate protein abundance, modification status, or subcellular localization.
- The protein complex repository remains incomplete for novel or condition-specific complexes; CORUM alone contributes only 694 human complexes of size ≥5 proteins.
- Tissue-specific annotations were derived from population-level data and may not fully reflect interpatient heterogeneity.

## Evidence

- [abstract] Protrec2's HeLa upper-bound benchmark performance (96.5% recovery, 453 proteins validated) that the equal-prior ablation must match or exceed.: "achieving the highest recovery rates: up to 98.4% in A549 and 96.5% in HeLa and validating 650 and 453 proteins, respectively."
- [other] Unified-prior ablation outcome on the Astral HeLa dataset: Protrec2 keeps its advantage even when baselines receive the expanded complex set.: [evidence span withheld — anchor status: not_found]
- [discussion] Probability cutoff range scanned when computing recovery curves.: "Protrec2 demonstrates monotonic improvement from cutoff 0.50 to 0.95"
- [discussion] Scope of the baseline comparison: upper- and lower-bound benchmarks on A549 and HeLa proteomes.: "Protrec2 consistently and robustly outperforms baseline methods across upper- and lower-bound benchmarks on A549 and HeLa proteomes, demonstrating platform-agnostic performance."
- [readme] README-reported performance figures against which the ablation reproduction is checked.: [evidence span withheld — anchor status: external_doc]
- [intro] CORUM coverage limitation that motivates using the expanded multi-source complex set in the ablation.: "as CORUM only contains 694 human protein complexes above the size threshold (≥5 proteins), this limits its coverage of the broader human complexome and proteome"
