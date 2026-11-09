---
name: multiple-testing-pvalue-correction
description: Use when calling RBP target transcripts from differential RNA-editing
  statistics where thousands of 50-nt transcript bins are tested simultaneously and
  raw Poisson P values would inflate false target calls.
when_to_use_negative:
- When editing sites have not passed the Poisson filter (maximum p-value 0.1 default)
  and coverage threshold of 10 — correction on unfiltered sites yields spuriously
  inflated test counts
- When the analysis goal is per-site editing detection (callediting) rather than transcript-level
  target calling, where bin-level adjustment is not applicable
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: Mapit calltargets
  role: Performs the differential editing analysis in which adjusted P values are
    computed at gene/transcript level for target calling
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: conda
  role: Recommended installation channel for the MAPIT-seq pipeline software that
    runs calltargets
- name: Snakemake
  role: Workflow manager used to orchestrate the MAPIT-seq pipeline steps including
    calltargets
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:833ca5d3d71328fbaedce11bf8ac1c1e23fb5d7aca1ef340fca6f2676c9e6612
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WangLabPKU/MAPIT-seq
  repo_url: https://github.com/WangLabPKU/MAPIT-seq
  tools:
  - Mapit calltargets
  - Snakemake
  - conda
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# multiple-testing-pvalue-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Adjust per-test P values for multiplicity during MAPIT-seq calltargets differential-editing analysis so that RBP target transcripts (e.g., PRC2 components EED, EZH2, SUZ12) are called using adjusted P < 0.05 rather than raw editing-site P values.

## When to use

Use when calling RBP target transcripts from differential RNA-editing statistics where thousands of 50-nt transcript bins are tested simultaneously and raw Poisson P values would inflate false target calls.

## When NOT to use

- When editing sites have not passed the Poisson filter (maximum p-value 0.1 default) and coverage threshold of 10 — correction on unfiltered sites yields spuriously inflated test counts
- When the analysis goal is per-site editing detection (callediting) rather than transcript-level target calling, where bin-level adjustment is not applicable

## Inputs

- Per-sample RNA editing event table from the callediting step (*_Edit_*_DATA.tsv under 6-Edit_calling)
- Differential editing statistics from Mapit calltargets (editing fold enrichment, MAPIT score, per-bin Poisson P values)
- Bin size and coverage parameters (binSize 50, minimum editing-site coverage 10)

## Outputs

- Gene/transcript-level adjusted P values per RBP (e.g., XIST: 0.0463 EED, 1.72e-3 EZH2, 9.94e-6 SUZ12)
- Significant RBP target gene lists per component under adjusted P < 0.05
- Summary CSV such as prc2_target_adjusted_pvalues.csv

## How to apply

After running the Mapit calltargets differential-editing step on the *_Edit_*_DATA.tsv editing tables produced by the callediting step, split transcripts into 50-nt bins (binSize 50) with minimum editing-site coverage of 10, and apply the Poisson editing-site filter (default maximum p-value 0.1, tightened to the reported significance level). Compute adjusted P values at the gene/transcript level to account for the many bins tested per RBP, then call targets with editing fold enrichment > 2, MAPIT score > 0.5, and adjusted P < 0.05. The rationale is that raw site-level P values cannot be compared across bins without correction; only after adjustment do near-empty target sets (as for PRC2 components, where XIST adjusted P values were 0.0463 for EED, 1.72 × 10−3 for EZH2 and 9.94 × 10−6 for SUZ12) become trustworthy evidence of weak binding. Write the reproduced adjusted P values and target lists to a CSV (e.g., prc2_target_adjusted_pvalues.csv) and validate against reported values within numerical tolerance.

## Related tools

- **Mapit calltargets** (Performs the differential editing analysis in which adjusted P values are computed at gene/transcript level for target calling) — https://github.com/WangLabPKU/MAPIT-seq
- **conda** (Recommended installation channel for the MAPIT-seq pipeline software that runs calltargets)
- **Snakemake** (Workflow manager used to orchestrate the MAPIT-seq pipeline steps including calltargets)

## Examples

```
mapit calltargets --input 6-Edit_calling/*_Edit_*_DATA.tsv --binSize 50 --coverage 10 --pvalue 0.1
```

## Evaluation signals

- Reproduced XIST adjusted P values match the reported values (0.0463 for EED, 1.72 × 10−3 for EZH2, 9.94 × 10−6 for SUZ12) within numerical tolerance
- PRC2 target gene sets for EED, EZH2 and SUZ12 remain near-empty after adjustment, with XIST the only common target
- Every called target satisfies all three thresholds: editing fold enrichment > 2, MAPIT score > 0.5, adjusted P < 0.05
- Adjusted P values are monotonically ordered relative to raw P values and no adjusted value falls below its raw counterpart
- Output CSV (prc2_target_adjusted_pvalues.csv) contains adjusted P values and target lists for all three components

## Limitations

- Near-empty target sets can reflect genuinely weak PRC2–RNA binding rather than pipeline failure, so low target counts should not be treated as an error signal
- Results depend on the Poisson filter setting (default maximum p-value 0.1) and minimum coverage of 10; tightening these changes the test set and hence the adjusted values
- Adjustment is performed at gene/transcript level after binning (binSize 50); per-bin raw P values are not directly comparable to reported adjusted values

## Evidence

- [other] XIST as the only common target with reported adjusted P values: [evidence span withheld — anchor status: not_found]
