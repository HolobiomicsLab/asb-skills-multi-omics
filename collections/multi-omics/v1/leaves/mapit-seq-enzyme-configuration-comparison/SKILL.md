---
name: mapit-seq-enzyme-configuration-comparison
description: Use when MAPIT-seq editing calls exist for both dual-editor and single-deaminase
  (hADAR2dd-only or rAPOBEC1-only) samples and you need to measure which enzyme configuration
  recovers more known RBP targets.
when_to_use_negative:
- No control (non-enzyme-expressing) sample is available, since calltargets requires
  treatName/controlName differential editing comparison
- The goal is identifying novel RBP targets rather than benchmarking sensitivity against
  published target sets
- Editing event tables come from a non-MAPIT-seq pipeline (e.g. plain REDItools2 output)
  incompatible with the calltargets binning/Poisson parameters
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3524
tools:
- name: MAPIT-seq
  role: Pipeline providing 'Mapit callediting' and 'Mapit calltargets' for differential
    editing analysis and RBP target calling per enzyme configuration
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: GATK
  role: RNA editing detection engine underlying the callediting step that produces
    the editing event tables
- name: REDItools2
  role: Optimized RNA editing calling tool within the MAPIT-seq editing-detection
    ecosystem
  repo: https://github.com/BioinfoUNIBA/REDItools2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:6e33a5f097c69e09101a31c10bcaa39734eb08d660071f9c31295cf120aa7c2c
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
  - MAPIT-seq
  - REDItools2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# mapit-seq-enzyme-configuration-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify the sensitivity gain of the MAPIT-seq dual-editor fusion construct (rAPOBEC1–pAG–hADAR2dd) over single-deaminase configurations by comparing recovery of published RBP targets from 'Mapit calltargets' output, reproducing the paper's reported ~20–30% improvement.

## When to use

Use when MAPIT-seq editing calls exist for both dual-editor and single-deaminase (hADAR2dd-only or rAPOBEC1-only) samples and you need to measure which enzyme configuration recovers more known RBP targets.

## When NOT to use

- No control (non-enzyme-expressing) sample is available, since calltargets requires treatName/controlName differential editing comparison
- The goal is identifying novel RBP targets rather than benchmarking sensitivity against published target sets
- Editing event tables come from a non-MAPIT-seq pipeline (e.g. plain REDItools2 output) incompatible with the calltargets binning/Poisson parameters

## Inputs

- RNA editing event table from 'Mapit callediting' (e.g. RBP_Edit_GE_RPE_DATA.tsv) for dual-editor and single-deaminase MAPIT-seq samples
- GRCh38 transcript annotation (-v GRCh38 for calltargets)
- Published RBP target sets from Extended Data Fig. 2b,c and Supplementary Table 3

## Outputs

- Called RBP target transcript lists per enzyme configuration
- target_recovery_comparison.csv with per-configuration target recovery rates and percent gain of dual-editor over single editors

## How to apply

Load the RNA editing event tables (e.g. RBP_Edit_GE_RPE_DATA.tsv) produced by 'Mapit callediting' for the dual-editor and each single-deaminase MAPIT-seq sample. Run 'Mapit calltargets' (-v GRCh38) separately per construct, setting --treatName/--controlName to compare enzyme-expressing vs control samples, with transcript binning at binSize 50, minimum coverage 10 for valid editing sites, and a Poisson filter at -p 0.05 on editing sites (report-level significance -p 0.05). Intersect the called RBP target transcripts from each configuration with the published target sets from Extended Data Fig. 2b,c and Supplementary Table 3 to compute per-configuration recovery rates. Finally, calculate the percent gain in published-target recovery of the dual-editor construct relative to each single editor and write target_recovery_comparison.csv; validation requires the recomputed gain to fall within the reported 20–30% range.

## Related tools

- **MAPIT-seq** (Pipeline providing 'Mapit callediting' and 'Mapit calltargets' for differential editing analysis and RBP target calling per enzyme configuration) — https://github.com/WangLabPKU/MAPIT-seq
- **GATK** (RNA editing detection engine underlying the callediting step that produces the editing event tables)
- **REDItools2** (Optimized RNA editing calling tool within the MAPIT-seq editing-detection ecosystem) — https://github.com/BioinfoUNIBA/REDItools2

## Examples

```
Mapit calltargets -v GRCh38 --treatName dual_editor --controlName control --binSize 50 -c 10 -p 0.05
```

## Evaluation signals

- Recomputed percent gain in published-target recovery for the dual-editor construct falls within the reported 20–30% range
- Each calltargets run uses the specified parameters: binSize 50, minimum coverage 10, Poisson filter -p 0.05, report-level significance -p 0.05, -v GRCh38
- Per-configuration target recovery rates are computed against the same published target sets (Extended Data Fig. 2b,c and Supplementary Table 3)
- target_recovery_comparison.csv contains recovery rates for both single editors and the dual editor plus the percent gain

## Limitations

- Sensitivity gain is measured only against published RBP targets; the dual editor's advantage on novel targets is not established by this comparison
- The dual-editor construct must have been selected on the basis of its robust signals — the comparison presumes the specific rAPOBEC1–pAG–hADAR2dd fusion
- Editing-site Poisson filter default is 0.1; the reported benchmark used -p 0.05, so default settings will not reproduce the published numbers

## Evidence

- [supplementary] Dual-editor sensitivity gain: [evidence span withheld — anchor status: not_found]
