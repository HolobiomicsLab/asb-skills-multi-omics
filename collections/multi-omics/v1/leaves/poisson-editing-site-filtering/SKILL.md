---
name: poisson-editing-site-filtering
description: Use when calling RBP target transcripts from MAPIT-seq RNA editing event
  tables (e.g. *_Edit_*_DATA.tsv from callediting) and noisy editing sites must be
  pruned by Poisson p-value before differential enrichment testing.
when_to_use_negative:
- When the input is raw sequencing data rather than a callediting-derived editing
  event table — mapping and GATK RNA editing detection must run first
- When performing only edit-site detection or high-confidence edit cluster identification
  (hc_cluster) rather than differential target calling — the Poisson filter is a calltargets
  parameter
- When comparing editing between samples without an enzyme-expressing/control design,
  since the Poisson filter operates within the treatment-vs-control differential framework
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: MAPIT-seq (Mapit calltargets)
  role: Provides the Poisson filter (-p/--pvalue) applied to editing sites during
    differential editing target calling
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: GATK
  role: Upstream callediting step that produces the RNA editing event tables the Poisson
    filter is applied to
- name: REDItools2
  role: Alternative RNA editing event detection tool in the MAPIT-seq pipeline whose
    output feeds the target-calling filters
  repo: https://github.com/BioinfoUNIBA/REDItools2
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:3613c86f2b3ee94530d33ad76eef04add99052176b670e51af8aa47a9974fb8e
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
  - GATK
  - MAPIT-seq (Mapit calltargets)
  - REDItools2
  - conda
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# poisson-editing-site-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A statistical filter in the MAPIT-seq 'calltargets' differential-editing step that rejects RNA editing sites failing a Poisson significance test before transcript-level RBP target calling, applied alongside minimum-coverage and transcript-binning thresholds.

## When to use

Use when calling RBP target transcripts from MAPIT-seq RNA editing event tables (e.g. *_Edit_*_DATA.tsv from callediting) and noisy editing sites must be pruned by Poisson p-value before differential enrichment testing.

## When NOT to use

- When the input is raw sequencing data rather than a callediting-derived editing event table — mapping and GATK RNA editing detection must run first
- When performing only edit-site detection or high-confidence edit cluster identification (hc_cluster) rather than differential target calling — the Poisson filter is a calltargets parameter
- When comparing editing between samples without an enzyme-expressing/control design, since the Poisson filter operates within the treatment-vs-control differential framework

## Inputs

- RNA editing event table (e.g. RBP_Edit_GE_RPE_DATA.tsv or *_Edit_*_DATA.tsv) from the MAPIT-seq callediting step
- Treated (enzyme-expressing) and control sample identifiers for --treatName/--controlName
- Genome/transcript annotation for GRCh38 (-v GRCh38)

## Outputs

- Filtered editing-site table retaining only sites passing the Poisson p-value cutoff
- Per-RBP called target transcript list with editing fold enrichment, MAPIT score, and adjusted P values
- Summary tables such as prc2_target_adjusted_pvalues.csv or target_recovery_comparison.csv

## How to apply

Run the 'Mapit calltargets' differential editing analysis on the per-sample editing event table produced by the callediting (GATK) step, splitting transcripts into 50-nt bins (binSize 50) and requiring minimum coverage of 10 reads for valid editing sites. Apply the Poisson filter with --pvalue (-p): the tool default is 0.1, but the published PRC2 and dual-editor sensitivity analyses tighten it to 0.05 for editing-site significance, with a further report-level significance threshold of 0.05 for adjusted P values. Set --treatName/--controlName to the enzyme-expressing vs control samples for each construct configuration (single hADAR2dd, single rAPOBEC1, or dual-editor fusion). Targets are then called using editing fold enrichment > 2 and MAPIT score > 0.5 with adjusted P < 0.05; the rationale is that only sites whose editing counts deviate significantly from background expectation should contribute to the transcript-level binding signal.

## Related tools

- **MAPIT-seq (Mapit calltargets)** (Provides the Poisson filter (-p/--pvalue) applied to editing sites during differential editing target calling) — https://github.com/WangLabPKU/MAPIT-seq
- **GATK** (Upstream callediting step that produces the RNA editing event tables the Poisson filter is applied to)
- **REDItools2** (Alternative RNA editing event detection tool in the MAPIT-seq pipeline whose output feeds the target-calling filters) — https://github.com/BioinfoUNIBA/REDItools2

## Examples

```
Mapit calltargets --treatName RBP --controlName Ctrl -v GRCh38 --binSize 50 -c 10 -p 0.05
```

## Evaluation signals

- Reproduced XIST adjusted P values for EED, EZH2 and SUZ12 match reported 0.0463, 1.72 × 10⁻³ and 9.94 × 10⁻⁶ within numerical tolerance
- Called PRC2 target sets remain near-empty, supporting weak PRC2–RNA binding
- Dual-editor target recovery over single-deaminase configurations falls within the reported ~20–30% gain (Extended Data Fig. 2b,c, Supplementary Table 3)
- All retained editing sites have coverage ≥ 10 and Poisson p-values at or below the specified cutoff (0.05, or 0.1 default)
- Output CSV schema contains per-transcript adjusted P values and fold enrichment values consistent with filtering thresholds

## Limitations

- The Poisson filter's default is 0.1; using the default rather than the published 0.05 threshold can change target call sets and PRC2 near-empty results
- Filtering depends on minimum coverage of 10 for valid editing sites — low-coverage transcripts lose usable sites entirely
- Tool documentation lists thresholds as tunable parameters, so results are sensitive to binSize and p-value choices

## Evidence

- [other] Poisson filter of -p 0.05 on editing sites (report-level significance -p 0.05): [evidence span withheld — anchor status: not_found]
- [results] XIST adjusted P values were 0.0463 (EED), 1.72 × 10−3 (EZH2) and 9.94 × 10−6 (SUZ12): [evidence span withheld — anchor status: not_found]
