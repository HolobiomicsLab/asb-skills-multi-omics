---
name: logit-space-equal-weight-evidence-averaging
description: Use when recovering missing proteins in MS proteomics by combining multiple
  probabilistic evidence sources (prior, co-complex support, complex confidence) and
  you need a stable averaging rule that resists domination by extreme values.
when_to_use_negative:
- The analysis requires protein abundance, modification status, or subcellular localization
  — Protrec2 operates on a binary presence/absence basis and does not incorporate
  these
- The target condition involves novel or condition-specific complexes absent from
  the protein complex repository, or tissue priors that must reflect interpatient
  heterogeneity (annotations are population-level)
- The task is general differential-expression or enrichment testing rather than recovering
  unreported but biologically present proteins from a missing-value-laden proteomics
  matrix
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0128
tools:
- name: Protrec2
  role: Implements the Bayesian framework that logit-space averages prior, co-complex,
    and complex-confidence evidence to update unobserved protein presence probabilities
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Runtime for main.py, which accepts --expression, --output_dir, --tissue, --fdr,
    and --threshold parameters
- name: pip
  role: Installs dependencies via 'pip install -r requirements.txt' before running
    the inference pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:696eb8331b223ff2a7658b683e10c96491856078360296b14ed4ddc4574e98e4
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
  - pip
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# logit-space-equal-weight-evidence-averaging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Protrec2's Bayesian evidence-combination scheme: prior detection probability, co-complex structural support, and tissue-specific complex confidence are averaged with equal weights (alpha = beta = gamma = 1/3) in logit space, then mapped back to probability via the sigmoid, stabilizing inference so extreme values do not dominate.

## When to use

Use when recovering missing proteins in MS proteomics by combining multiple probabilistic evidence sources (prior, co-complex support, complex confidence) and you need a stable averaging rule that resists domination by extreme values.

## When NOT to use

- The analysis requires protein abundance, modification status, or subcellular localization — Protrec2 operates on a binary presence/absence basis and does not incorporate these
- The target condition involves novel or condition-specific complexes absent from the protein complex repository, or tissue priors that must reflect interpatient heterogeneity (annotations are population-level)
- The task is general differential-expression or enrichment testing rather than recovering unreported but biologically present proteins from a missing-value-laden proteomics matrix

## Inputs

- Expression matrix CSV (rows: proteins with UniProt ID or symbol; columns: samples; non-negative numeric values), e.g. example/example_expression.csv
- Built-in tissue-specific complex prior file updated_tissue_complex_*.csv for the selected tissue (e.g. 'lung')
- Protein complex list (CORUM, Complex Portal, Reactome, STRING-expanded complexes, 3,885 unique complexes)
- Parameters: --fdr (0.01), --threshold (5), --tissue

## Outputs

- Per-sample CSV files (e.g. results/PROTREC2_result_{sample_name}.csv) with header 'Protein,Probability' and one row per recovered protein
- Recovered protein probabilities in [0, 1], z-score and min-max normalized, filtered at the 0.95 confidence cutoff

## How to apply

Initialize protein priors so observed proteins receive 1 - FDR (e.g. --fdr 0.01) and unobserved proteins receive 0.1. For each unobserved protein, compute the co-complex conditional probability from observed proteins co-occurring in tissue-annotated complexes passing the size threshold (--threshold 5), using the built-in tissue-specific prior (updated_tissue_complex_*.csv, e.g. tissue 'lung'). Combine the three evidence layers with equal weights alpha = beta = gamma = 1/3 by averaging in logit space, then apply the sigmoid to map the smoothed logit score back to probability space. The logit-space smoothing is the rationale: it stabilizes the Bayesian update so extreme probability values do not dominate, enabling nuanced recovery under sparse or noisy data. Finally z-score-normalize and min-max-normalize the recovered probabilities and retain proteins meeting the confidence cutoff (up to 0.95).

## Related tools

- **Protrec2** (Implements the Bayesian framework that logit-space averages prior, co-complex, and complex-confidence evidence to update unobserved protein presence probabilities) — https://github.com/miaomiao6606/Protrec2
- **Python** (Runtime for main.py, which accepts --expression, --output_dir, --tissue, --fdr, and --threshold parameters)
- **pip** (Installs dependencies via 'pip install -r requirements.txt' before running the inference pipeline)

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- Each output CSV exists, parses with columns 'Protein' and 'Probability', and all probabilities lie within [0, 1]
- Retained proteins are only those whose normalized confidence score is at or above the 0.95 cutoff
- Extreme input probability values (e.g. prior 0.1 for unobserved, 1 - FDR = 0.99 for observed) do not collapse the update — the sigmoid of the equal-weight logit average stays bounded and interpretable
- Unobserved proteins get non-trivial co-complex conditional probabilities only via observed partners in tissue-annotated complexes of size >= the --threshold 5 cutoff
- Benchmark recovery rates are comparable to reported values (e.g. 98.4% in A549, 96.5% in HeLa upper-bound; >85% CPTAC support in lung tumor-normal pairs)

## Limitations

- Tissue-specific annotations were derived from population-level data and might not fully reflect interpatient heterogeneity
- Protrec2 operates on a binary presence/absence basis and does not yet incorporate protein abundance, modification status, or subcellular localization
- The protein complex repository, while comprehensive, remains incomplete for novel or condition-specific complexes
- Validation performance is constrained by proteomics dataset coverage, with reported recovery rates representing lower bounds on true performance

## Evidence

- [readme] Output format specification: [evidence span withheld — anchor status: external_doc]
