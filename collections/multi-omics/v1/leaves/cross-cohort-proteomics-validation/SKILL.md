---
name: cross-cohort-proteomics-validation
description: Use when recovered/predicted missing proteins from a discovery proteomics
  run (e.g. Protrec2 output on six matched lung tumor-normal pairs from PXD000603)
  need independent confirmation in a public cohort such as CPTAC before being reported.
when_to_use_negative:
- No independent external cohort covering the tissue/cancer type is available, so
  recovered proteins cannot be cross-validated (reported support rates would be meaningless
  lower bounds).
- The recovered-protein outputs are probability lists only and no tumor/normal expression
  values exist, so the expression-direction consistency check cannot be performed.
- The goal is de novo missing-protein recovery rather than validation of already-recovered
  proteins — use the Protrec2 recovery skill instead.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Protrec2
  role: Produces the recovered missing-protein probability lists (cancer-specific
    and normal-exclusive) that this skill validates against CPTAC cohorts
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Used to binarize recovered-protein probabilities, intersect them with CPTAC
    cohort protein lists, and compute support and direction-consistency rates
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:92c5f3a031e55a712f049e2851cde475cf41f9cd583a6aac6b8be600b170cb83
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cross-cohort-proteomics-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Independent validation of Protrec2-recovered missing proteins by checking them against external large-scale clinical proteomic cohorts (CPTAC lung datasets PDC000234 and PDC000153), computing per-class support rates and tumor-vs-normal expression-direction consistency.

## When to use

Use when recovered/predicted missing proteins from a discovery proteomics run (e.g. Protrec2 output on six matched lung tumor-normal pairs from PXD000603) need independent confirmation in a public cohort such as CPTAC before being reported.

## When NOT to use

- No independent external cohort covering the tissue/cancer type is available, so recovered proteins cannot be cross-validated (reported support rates would be meaningless lower bounds).
- The recovered-protein outputs are probability lists only and no tumor/normal expression values exist, so the expression-direction consistency check cannot be performed.
- The goal is de novo missing-protein recovery rather than validation of already-recovered proteins — use the Protrec2 recovery skill instead.

## Inputs

- Per-sample Protrec2 output CSVs (Protein,Probability rows) from six matched lung tumor-normal pairs (PXD000603 expression matrix)
- CPTAC lung cancer proteomic datasets PDC000234 and PDC000153
- Binarization cutoffs in the 0.50-0.95 probability range

## Outputs

- CPTAC support rate per recovered-protein class (fraction of recovered proteins observed in PDC000234/PDC000153)
- Expression-direction consistency percentages (tumor up/down vs normal up/down) for cancer-specific and normal-exclusive recovered proteins
- Validated list of recovered proteins supported by the independent cohorts

## How to apply

First run Protrec2 (`python main.py --expression <matrix> --tissue lung --fdr 0.01 --threshold 5`) and binarize the per-sample recovered-protein probability CSVs at probability cutoffs in the 0.50-0.95 range. Classify proteins recovered in tumors as cancer-specific and those recovered only in normals as normal-exclusive, then intersect each class against the independent CPTAC lung cancer proteomic datasets PDC000234 and PDC000153, computing the fraction observed there (target: 87.18% cancer-specific, 85.45% normal-exclusive). As a second orthogonal check, compare recovered-protein expression directions between tumor and normal samples: cancer-specific proteins should be predominantly upregulated (~68.5%) and normal-exclusive proteins predominantly downregulated (~90.0%). Accept the run when support rates match the reported values within tolerance and at least 85% of predicted proteins are CPTAC-supported.

## Related tools

- **Protrec2** (Produces the recovered missing-protein probability lists (cancer-specific and normal-exclusive) that this skill validates against CPTAC cohorts) — https://github.com/miaomiao6606/Protrec2
- **Python** (Used to binarize recovered-protein probabilities, intersect them with CPTAC cohort protein lists, and compute support and direction-consistency rates)

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- CPTAC support rate ≈ 87.18% for cancer-specific recovered proteins and ≈ 85.45% for normal-exclusive recovered proteins (within tolerance).
- At least 85% of Protrec2-predicted proteins are supported by CPTAC PDC000234/PDC000153.
- ≈68.5% of cancer-specific recovered proteins are upregulated in tumors; ≈90.0% of normal-exclusive recovered proteins are downregulated.
- Support improves monotonically as the binarization cutoff increases from 0.50 to 0.95, consistent with the paper's threshold sensitivity analysis.

## Limitations

- Protrec2 operates on a binary presence/absence basis and does not incorporate protein abundance, modification status, or subcellular localization.
- Tissue-specific annotations were derived from population-level data and may not reflect interpatient heterogeneity.
- Validation performance is constrained by proteomics dataset coverage, so reported recovery/support rates are lower bounds on true performance.

## Evidence

- [readme] Probability: Recovery probability (0-1): [evidence span withheld — anchor status: external_doc]
