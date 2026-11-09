---
name: protein-expression-direction-consistency-analysis
description: Use when recovered/predicted missing proteins from tumor-normal proteomic
  data (e.g., Protrec2 per-sample probability CSVs binarized at 0.50-0.95 cutoffs)
  must be checked for direction consistency against independent cohort expression
  (e.g., CPTAC PDC000234/PDC000153).
when_to_use_negative:
- Input is raw MS spectra or an unprocessed expression matrix — recovery must run
  first (Protrec2 main.py) before direction consistency can be assessed
- The study has no matched tumor-normal design or no independent validation cohort,
  so direction comparison between phenotypes is undefined
- Proteins of interest are already observed (not missing/recovered), since this skill
  evaluates recovered presence predictions rather than quantified abundances
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Protrec2
  role: Produces the per-sample recovered-protein probability CSVs whose cancer-specific/normal-exclusive
    classes are checked for direction consistency
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Used to binarize probabilities at 0.50-0.95 cutoffs, classify proteins, and
    compute CPTAC support and up/down direction fractions
- name: pip
  role: Installs Protrec2 dependencies from requirements.txt before running the recovery-then-validation
    workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:ae2f4ea2fc424430195ada07d1af957ffa5a7a31c4f78feac5a4182b190a4440
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# protein-expression-direction-consistency-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates biologically recovered missing proteins (e.g., Protrec2 outputs on lung tumor-normal pairs) by checking whether their expression directions are consistent with clinical expectations: cancer-specific proteins should be predominantly upregulated in tumors (~68.5%) and normal-exclusive proteins predominantly downregulated (~90.0%).

## When to use

Use when recovered/predicted missing proteins from tumor-normal proteomic data (e.g., Protrec2 per-sample probability CSVs binarized at 0.50-0.95 cutoffs) must be checked for direction consistency against independent cohort expression (e.g., CPTAC PDC000234/PDC000153).

## When NOT to use

- Input is raw MS spectra or an unprocessed expression matrix — recovery must run first (Protrec2 main.py) before direction consistency can be assessed
- The study has no matched tumor-normal design or no independent validation cohort, so direction comparison between phenotypes is undefined
- Proteins of interest are already observed (not missing/recovered), since this skill evaluates recovered presence predictions rather than quantified abundances

## Inputs

- Per-sample Protrec2 output CSVs (Protein,Probability rows) from six matched lung tumor-normal pairs (PXD000603 expression matrix)
- Independent CPTAC lung cancer proteomic datasets PDC000234 and PDC000153
- Binarization probability cutoff in the 0.50-0.95 range

## Outputs

- Cancer-specific and normal-exclusive protein class assignments per sample pair
- CPTAC support rates per class (target: 87.18% cancer-specific, 85.45% normal-exclusive)
- Expression direction concordance fractions (target: ~68.5% upregulated cancer-specific, ~90.0% downregulated normal-exclusive)

## How to apply

First run the recovery pipeline (Protrec2 with --tissue lung, --fdr 0.01, --threshold 5) to obtain per-sample CSVs of Protein,Probability rows. Binarize recovered proteins at probability cutoffs in the 0.50-0.95 range and partition them into cancer-specific (recovered in tumors) and normal-exclusive (recovered only in normals) classes. Validate each class against the independent CPTAC lung cohorts PDC000234 and PDC000153, keeping only CPTAC-supported proteins. Then compare recovered-protein expression directions between tumor and normal samples, computing the fraction upregulated among cancer-specific and downregulated among normal-exclusive proteins; accept the run when these match the reported ~68.5% upregulated and ~90.0% downregulated and at least 85% of predicted proteins are CPTAC-supported (87.18% cancer-specific, 85.45% normal-exclusive). The rationale is that a recovery method that merely over-predicts would not reproduce directional, phenotype-consistent patterns in an independent clinical cohort.

## Related tools

- **Protrec2** (Produces the per-sample recovered-protein probability CSVs whose cancer-specific/normal-exclusive classes are checked for direction consistency) — https://github.com/miaomiao6606/Protrec2
- **Python** (Used to binarize probabilities at 0.50-0.95 cutoffs, classify proteins, and compute CPTAC support and up/down direction fractions)
- **pip** (Installs Protrec2 dependencies from requirements.txt before running the recovery-then-validation workflow)

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- CPTAC support rate for cancer-specific recovered proteins equals 87.18% (and normal-exclusive 85.45%) within tolerance
- At least 85% of predicted proteins are independently supported by CPTAC cohorts PDC000234 and PDC000153
- Cancer-specific proteins are predominantly upregulated in tumors at approximately 68.5%
- Normal-exclusive proteins are predominantly downregulated in tumors at approximately 90.0%
- Class assignment is reproducible across probability cutoffs in the monotonic 0.50-0.95 range

## Limitations

- Tissue-specific annotations were derived from population-level data and might not fully reflect interpatient heterogeneity
- Protrec2 operates on a binary presence/absence basis and does not incorporate protein abundance, modification status, or subcellular localization, so direction analysis relies on externally observed expression values
- Validation performance is constrained by proteomics dataset coverage, with reported recovery rates representing lower bounds on true performance

## Evidence
