---
name: co-complex-conditional-probability-scoring
description: Use when recovering missing (unobserved) proteins from an MS proteomics
  expression matrix by scoring each candidate's co-complex support against tissue-specific
  complex annotations before Bayesian combination.
when_to_use_negative:
- The goal is differential abundance testing on observed proteins rather than recovering
  unreported/missing proteins — Protrec2 operates on a binary presence/absence basis
  and does not incorporate protein abundance, modification status, or subcellular
  localization
- Protein complexes of interest cannot be reliably annotated for the tissue or are
  novel/condition-specific, since the complex repository remains incomplete for novel
  or condition-specific complexes
- The input is already a scored probability table rather than a raw expression matrix
  with missing proteins to recover
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0128
tools:
- name: Protrec2
  role: Implements the co-complex conditional probability computation and Bayesian
    logit-space combination for missing protein recovery
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Runtime for main.py, which takes the expression matrix, tissue, FDR, and threshold
    parameters
- name: pip
  role: Installs dependencies from requirements.txt before running the pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:49d0650c9a5eb0e40b800553e82b66cc1f1919a3f354d6449fb8fe6e1cb49b53
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
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# co-complex-conditional-probability-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes, for each unobserved protein, a conditional probability of presence from observed proteins that co-occur with it in tissue-annotated complexes passing a size threshold, which Protrec2 then combines with the prior and complex-level confidence via logit-space averaging.

## When to use

Use when recovering missing (unobserved) proteins from an MS proteomics expression matrix by scoring each candidate's co-complex support against tissue-specific complex annotations before Bayesian combination.

## When NOT to use

- The goal is differential abundance testing on observed proteins rather than recovering unreported/missing proteins — Protrec2 operates on a binary presence/absence basis and does not incorporate protein abundance, modification status, or subcellular localization
- Protein complexes of interest cannot be reliably annotated for the tissue or are novel/condition-specific, since the complex repository remains incomplete for novel or condition-specific complexes
- The input is already a scored probability table rather than a raw expression matrix with missing proteins to recover

## Inputs

- Expression matrix CSV (rows: proteins as UniProt IDs or gene symbols; columns: samples; non-negative numeric values, e.g. example/example_expression.csv)
- Tissue-specific complex prior built-in from updated_tissue_complex_*.csv (e.g. tissue 'lung')
- Parameters: --fdr 0.01, --threshold 5 (minimum complex size)

## Outputs

- Per-sample CSV files (e.g. results/PROTREC2_result_{sample_name}.csv) with header 'Protein,Probability' and one row per recovered protein meeting the 0.95 confidence cutoff
- Co-complex conditional probabilities for unobserved proteins feeding the Bayesian update

## How to apply

Load the expression matrix (rows: proteins, columns: samples, e.g. example/example_expression.csv) and the built-in tissue-specific complex prior (updated_tissue_complex_*.csv, e.g. tissue 'lung'). Initialize priors so observed proteins get 1 - FDR (default --fdr 0.01) and unobserved proteins get 0.1. For each unobserved protein, restrict to tissue-annotated complexes with size at least --threshold 5 and compute the co-complex conditional probability from the observed proteins that co-occur in those complexes. Combine this co-complex support with the prior detection probability and the tissue-specific complex confidence using equal weights alpha = beta = gamma = 1/3 via logit-space averaging (which stabilizes inference so extreme values do not dominate), then map back through the sigmoid. Apply z-score then min-max normalization and retain proteins with normalized confidence >= 0.95.

## Related tools

- **Protrec2** (Implements the co-complex conditional probability computation and Bayesian logit-space combination for missing protein recovery) — https://github.com/miaomiao6606/Protrec2
- **Python** (Runtime for main.py, which takes the expression matrix, tissue, FDR, and threshold parameters)
- **pip** (Installs dependencies from requirements.txt before running the pipeline)

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- Each output CSV (e.g. results/PROTREC2_result_{sample}.csv) exists and parses with columns 'Protein' and 'Probability'
- All reported probabilities are within [0, 1]
- Every retained protein has a normalized confidence score at or above the 0.95 cutoff
- Observed-protein priors equal 1 - FDR (0.99 at --fdr 0.01) and unobserved-protein priors equal 0.1 before inference
- Only complexes meeting the size threshold (--threshold 5) contribute to the co-complex conditional probability

## Limitations

- Tissue-specific annotations were derived from population-level data and might not fully reflect interpatient heterogeneity
- Protrec2 currently operates on a binary presence/absence basis and does not yet incorporate protein abundance, modification status, or subcellular localization
- The protein complex repository, while comprehensive, remains incomplete for novel or condition-specific complexes
- Memory errors can occur with large datasets; process samples in batches or reduce the complex size threshold

## Evidence

- [methods] For each unobserved protein, compute the co-complex conditional probability from observed proteins co-occurring in tissue-annotated complexes passing the size threshold (--threshold 5).: [evidence span withheld — anchor status: not_found]
- [discussion] Protrec2 employs a Bayesian inference model that integrates three complementary layers of evidence—prior detection probability, cocomplex structural support, and tissue-specific complex: "Protrec2 employs a Bayesian inference model that integrates three complementary layers of evidence—prior detection probability, cocomplex structural support, and tissue-specific complex"
- [discussion] This probabilistic formulation not only stabilizes inference through logit-space smoothing but also enables nuanced recovery under sparse or noisy data conditions: "This probabilistic formulation not only stabilizes inference through logit-space smoothing but also enables nuanced recovery under sparse or noisy data conditions"
- [readme] 1. Initialize protein probabilities based on observation status
2. For each protein, integrate:
   - Direct observation probability
   - Complex membership support
   - Tissue-specific prior probability: [evidence span withheld — anchor status: external_doc]
- [readme] python main.py \
  --expression example/example_expression.csv \
  --output_dir results/ \
  --tissue lung \
  --fdr 0.01 \
  --threshold 5: "--expression example/example_expression.csv \
  --output_dir results/ \
  --tissue lung \
  --fdr 0.01 \
  --threshold 5"
