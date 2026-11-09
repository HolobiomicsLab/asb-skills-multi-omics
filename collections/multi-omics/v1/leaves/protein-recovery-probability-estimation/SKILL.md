---
name: protein-recovery-probability-estimation
description: Use when a proteomics expression matrix (rows = proteins, columns = samples)
  has missing proteins you want to recover as per-sample probability scores, especially
  with matched tumor-normal samples and a known tissue of origin.
when_to_use_negative:
- The input is not an expression matrix but raw mass spectrometric data, sequence
  data, or a feature/abundance table needing preprocessing — Protrec2 requires a protein-by-sample
  CSV of non-negative values.
- The question concerns protein abundance levels, modification status, or subcellular
  localization — Protrec2 operates on a binary presence/absence basis and does not
  incorporate abundance or PTMs.
- The condition involves novel or condition-specific complexes not represented in
  CORUM/Complex Portal/Reactome/STRING, where the complex repository's coverage is
  insufficient.
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Protrec2
  role: 'Primary framework: integrates tissue-specific protein complex annotations
    with Bayesian inference to recover unreported but biologically present proteins
    and emit per-sample recovery probabilities'
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Runtime for the Protrec2 pipeline entry point (main.py) and for post-hoc binarization
    of probability outputs at cutoffs between 0.50 and 0.95
- name: pip
  role: Dependency installer used to provision the Protrec2 environment via requirements.txt
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:d0e0f70eb2afccac1b8fbd4cde61867da319fa193c01e708ac0d056fe83b6c49
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
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# protein-recovery-probability-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimate recovery probabilities for missing (unreported but likely present) proteins in mass spectrometry proteomics data using Protrec2, which integrates tissue-specific protein complex annotations with Bayesian inference over prior detection probability, cocomplex structural support, and tissue-specific complex confidence.

## When to use

Use when a proteomics expression matrix (rows = proteins, columns = samples) has missing proteins you want to recover as per-sample probability scores, especially with matched tumor-normal samples and a known tissue of origin.

## When NOT to use

- The input is not an expression matrix but raw mass spectrometric data, sequence data, or a feature/abundance table needing preprocessing — Protrec2 requires a protein-by-sample CSV of non-negative values.
- The question concerns protein abundance levels, modification status, or subcellular localization — Protrec2 operates on a binary presence/absence basis and does not incorporate abundance or PTMs.
- The condition involves novel or condition-specific complexes not represented in CORUM/Complex Portal/Reactome/STRING, where the complex repository's coverage is insufficient.

## Inputs

- Expression matrix CSV (rows: proteins as UniProt IDs or gene symbols; columns: sample names; values: non-negative numeric expression levels)
- Tissue type selection (e.g. lung) matched to built-in `updated_tissue_complex_*.csv` annotations covering 27 tissues
- Parameters: --tissue, --fdr (default 0.01), --threshold (default 5, minimum complex size)

## Outputs

- Per-sample CSV files `{output_dir}/PROTREC2_result_{sample_name}.csv` with columns Protein,Probability (recovery probability 0–1)
- Binarized recovered-protein sets at chosen probability cutoffs (0.50–0.95) for downstream classification and validation

## How to apply

Prepare the expression matrix as CSV with rows as protein identifiers (UniProt IDs or gene symbols) and columns as sample names, values non-negative. Install the Protrec2 package with `pip install -r requirements.txt` and run `python main.py` supplying `--expression`, `--output_dir`, `--tissue` (e.g. lung), `--fdr 0.01`, and `--threshold 5` (minimum complex size). The method loads 3,885 curated complexes from CORUM (size ≥ 5), Complex Portal, Reactome, and STRING-expanded interactions (confidence > 780), assigns tissue relevance scores (1.0 explicit support, 0.5 ambiguous, 0.0 non-relevant) plus GO-based functional coherence, then iteratively updates protein presence probabilities via Bayesian inference with logit-space smoothing. Binarize recovered proteins at probability cutoffs in the 0.50–0.95 range; downstream validation can compare predictions against independent cohorts such as CPTAC (PDC000234, PDC000153).

## Related tools

- **Protrec2** (Primary framework: integrates tissue-specific protein complex annotations with Bayesian inference to recover unreported but biologically present proteins and emit per-sample recovery probabilities) — https://github.com/miaomiao6606/Protrec2
- **Python** (Runtime for the Protrec2 pipeline entry point (main.py) and for post-hoc binarization of probability outputs at cutoffs between 0.50 and 0.95)
- **pip** (Dependency installer used to provision the Protrec2 environment via requirements.txt)

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- Output CSVs exist per sample at {output_dir}/PROTREC2_result_{sample_name}.csv with exactly the columns Protein and Probability, probabilities within 0–1.
- In benchmark upper-bound evaluations, recovery rates reach ~98.4% in A549 and ~96.5% in HeLa (650 and 453 proteins validated).
- In lower-bound evaluations, over 90% of predicted proteins are validated in A549 and 74.6% in HeLa.
- For lung tumor-normal applications, ≥85% of predicted proteins are supported by CPTAC cohorts (reported: 87.18% cancer-specific, 85.45% normal-exclusive).
- Expression-direction checks: cancer-specific recovered proteins predominantly upregulated (~68.5%) and normal-exclusive ones predominantly downregulated (~90.0%); probability cutoff sweep from 0.50 to 0.95 shows monotonic improvement.

## Limitations

- Protrec2 operates on binary presence/absence and does not incorporate protein abundance, modification status, or subcellular localization.
- Tissue-specific annotations were derived from population-level data and may not reflect interpatient heterogeneity.
- The protein complex repository is incomplete for novel or condition-specific complexes.
- Validation performance is constrained by proteomics dataset coverage; reported recovery rates are lower bounds on true performance.
- Memory errors can occur with large datasets — process samples in batches or reduce the complex size threshold.

## Evidence
