---
name: probability-normalization-and-confidence-thresholding
description: Use when a Bayesian protein-recovery run has produced raw presence probabilities
  for unobserved proteins and you must normalize scores across samples and apply a
  confidence cutoff (0.50–0.95, default 0.95) before reporting recovered proteins.
when_to_use_negative:
- You have raw MS peak intensities or an unnormalized expression matrix — this skill
  normalizes downstream protein presence probabilities, not raw proteomics signals
- You need proteins with moderate or weak evidence — a 0.95 cutoff discards scores
  in the 0.5–0.9 'moderate evidence' band that may still be biologically interesting
- The analysis requires protein abundance, modification status, or subcellular localization
  — Protrec2 operates on a binary presence/absence basis and its probabilities do
  not encode these
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Protrec2
  role: Generates the Bayesian protein presence probabilities that are z-score/min-max
    normalized and thresholded at the confidence cutoff
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Runs main.py which performs the normalization, cutoff filtering, and per-sample
    CSV output
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:dbd9dc66e17f82da9ab9c7c829efcbe611149d1fa31f027d64e4de29252fffaf
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# probability-normalization-and-confidence-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Post-Bayesian scoring step in Protrec2 that transforms raw recovered protein presence probabilities via z-score normalization followed by min-max normalization, then retains only proteins whose normalized confidence meets a 0.95 cutoff before writing per-sample results.

## When to use

Use when a Bayesian protein-recovery run has produced raw presence probabilities for unobserved proteins and you must normalize scores across samples and apply a confidence cutoff (0.50–0.95, default 0.95) before reporting recovered proteins.

## When NOT to use

- You have raw MS peak intensities or an unnormalized expression matrix — this skill normalizes downstream protein presence probabilities, not raw proteomics signals
- You need proteins with moderate or weak evidence — a 0.95 cutoff discards scores in the 0.5–0.9 'moderate evidence' band that may still be biologically interesting
- The analysis requires protein abundance, modification status, or subcellular localization — Protrec2 operates on a binary presence/absence basis and its probabilities do not encode these

## Inputs

- Raw recovered protein presence probabilities per unobserved protein from Protrec2 Bayesian inference (probabilities in [0, 1])
- Expression matrix CSV (rows: proteins as UniProt IDs or gene symbols, columns: samples), e.g. example/example_expression.csv
- Confidence cutoff parameter (0.50–0.95 range; 0.95 for strictest filtering)

## Outputs

- Per-sample CSV files (e.g. results/PROTREC2_result_{sample_name}.csv) with header 'Protein,Probability' and one row per recovered protein at or above the 0.95 cutoff
- Normalized confidence scores in [0, 1] for each retained protein

## How to apply

After Protrec2's Bayesian inference combines prior detection probability, co-complex structural support, and tissue-specific complex confidence via logit-space averaging (equal weights alpha = beta = gamma = 1/3, sigmoid mapping back to probability space), apply z-score normalization to the recovered protein probabilities followed by min-max normalization so scores are comparable across samples. Retain only proteins whose normalized confidence score is at or above the 0.95 cutoff; the paper's sensitivity analysis shows monotonic improvement from cutoff 0.50 to 0.95, so 0.95 is the strictest, highest-precision setting. Write one CSV per sample to the output directory (e.g. results/) with the header 'Protein,Probability' and one row per retained protein, with values constrained to [0, 1]. Interpretation bands: >0.9 strong evidence, 0.5–0.9 moderate, <0.5 weak evidence for presence.

## Related tools

- **Protrec2** (Generates the Bayesian protein presence probabilities that are z-score/min-max normalized and thresholded at the confidence cutoff) — https://github.com/miaomiao6606/Protrec2
- **Python** (Runs main.py which performs the normalization, cutoff filtering, and per-sample CSV output)

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- Each output CSV parses with exactly the columns 'Protein' and 'Probability'
- All reported Probability values fall within [0, 1]
- Every retained protein has a normalized confidence score at or above the 0.95 cutoff
- One CSV file exists per input sample, named per the output convention (e.g. PROTREC2_result_{sample_name}.csv)
- Sweeping the cutoff from 0.50 to 0.95 yields monotonic behavior in the number/quality of retained proteins, consistent with the paper's sensitivity analysis

## Limitations

- Protrec2 operates on a binary presence/absence basis; normalized probabilities do not incorporate protein abundance, modification status, or subcellular localization
- Tissue-specific priors were derived from population-level data and may not fully reflect interpatient heterogeneity
- The protein complex repository is incomplete for novel or condition-specific complexes, so probabilities for proteins outside well-curated complexes may be unreliable
- Reported recovery rates represent lower bounds on true performance because validation is constrained by proteomics dataset coverage

## Evidence

- [readme] Normalize and threshold final probabilities: [evidence span withheld — anchor status: external_doc]
- [readme] probability interpretation bands: [evidence span withheld — anchor status: not_found]
