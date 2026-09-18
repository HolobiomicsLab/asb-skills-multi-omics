---
name: target-recovery-benchmarking-against-published-reference
description: Use when comparing sensitivity of MAPIT-seq editor configurations (dual
  rAPOBEC1–pAG–hADAR2dd fusion vs. single hADAR2dd or rAPOBEC1) by intersecting called
  RBP target transcripts with published reference target sets (e.g., Extended Data
  Fig.
when_to_use_negative:
- When the goal is discovering novel RBP targets de novo rather than benchmarking
  recovery of an existing published reference set
- When editing event tables are not directly comparable across configurations (e.g.,
  differing --treatName/--controlName contrasts, coverage, or Poisson thresholds),
  since the percent-gain metric assumes identical filter parameters
- When no published target list exists for the RBP under study — the metric depends
  on intersecting with a reference set
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MAPIT-seq pipeline (Mapit calltargets / callediting)
  role: Calls differential editing sites and RBP target transcripts per editor configuration
    for the recovery benchmark
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: GATK
  role: RNA editing detection ('callediting') producing the editing event tables that
    feed target calling
- name: REDItools2
  role: RNA editing event detection tool in the MAPIT-seq ecosystem supporting the
    editing tables used here
  repo: https://github.com/BioinfoUNIBA/REDItools2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:b346f3b188a34e841a6145fff3546e8bbb27a29966c42cd167cc3d67c6fda7f1
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
  - MAPIT-seq pipeline (Mapit calltargets / callediting)
  - REDItools2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# target-recovery-benchmarking-against-published-reference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking procedure that quantifies the sensitivity of MAPIT-seq editor configurations (dual-editor rAPOBEC1–pAG–hADAR2dd vs. single-deaminase constructs) by measuring how many published RBP target transcripts each configuration recovers. The dual-editor fusion identifies approximately 20–30% more reported targets than either single editor alone, validating its selection on robustness grounds.

## When to use

Use when comparing sensitivity of MAPIT-seq editor configurations (dual rAPOBEC1–pAG–hADAR2dd fusion vs. single hADAR2dd or rAPOBEC1) by intersecting called RBP target transcripts with published reference target sets (e.g., Extended Data Fig. 2b,c; Supplementary Table 3) to compute percent gain in target recovery.

## When NOT to use

- When the goal is discovering novel RBP targets de novo rather than benchmarking recovery of an existing published reference set
- When editing event tables are not directly comparable across configurations (e.g., differing --treatName/--controlName contrasts, coverage, or Poisson thresholds), since the percent-gain metric assumes identical filter parameters
- When no published target list exists for the RBP under study — the metric depends on intersecting with a reference set

## Inputs

- RNA editing event table from MAPIT-seq callediting (e.g., RBP_Edit_GE_RPE_DATA.tsv) for dual-editor and single-deaminase samples
- Published RBP target reference sets (Extended Data Fig. 2b,c; Supplementary Table 3)
- GRCh38 transcript annotation (for transcript binning at binSize 50)

## Outputs

- Called RBP target transcript sets per editor configuration (from 'Mapit calltargets')
- Summary CSV of percent gain in published-target recovery: target_recovery_comparison.csv

## How to apply

Load the RNA editing event tables (e.g., RBP_Edit_GE_RPE_DATA.tsv) produced by 'Mapit callediting' for the dual-editor and single-deaminase MAPIT-seq samples. Run 'Mapit calltargets' (-v GRCh38) separately for each construct configuration, setting --treatName/--controlName to compare enzyme-expressing vs. control samples, with transcript binning at binSize 50, a minimum coverage of 10 for valid editing sites, and a Poisson filter of -p 0.05 on editing sites with report-level significance at p 0.05. Intersect the called RBP target transcripts from each configuration with the published target sets from Extended Data Fig. 2b,c and Supplementary Table 3 to compute per-configuration target recovery rates. Finally, calculate the percent gain in published-target recovery of the dual-editor construct relative to each single-deaminase configuration and write the comparison to target_recovery_comparison.csv. The differential-editing 'calltargets' step is what converts per-site editing events into RBP-bound transcript calls, so identical filter parameters must be applied across configurations for a fair sensitivity comparison.

## Related tools

- **MAPIT-seq pipeline (Mapit calltargets / callediting)** (Calls differential editing sites and RBP target transcripts per editor configuration for the recovery benchmark) — https://github.com/WangLabPKU/MAPIT-seq
- **GATK** (RNA editing detection ('callediting') producing the editing event tables that feed target calling)
- **REDItools2** (RNA editing event detection tool in the MAPIT-seq ecosystem supporting the editing tables used here) — https://github.com/BioinfoUNIBA/REDItools2

## Examples

```
Mapit calltargets -v GRCh38 --treatName dual_editor_rep --controlName control_rep --binSize 50 --coverage 10 -p 0.05
```

## Evaluation signals

- Recomputed percent gain in published-target recovery for the dual-editor construct over the single-deaminase configurations falls within the reported 20–30% range
- Identical calltargets parameters (binSize 50, minimum coverage 10, Poisson -p 0.05, report-level -p 0.05) are confirmed applied to every configuration before intersection
- Per-configuration target recovery rates computed against Extended Data Fig. 2b,c and Supplementary Table 3 reference sets are reproducible from the same RBP_Edit_GE_RPE_DATA.tsv input
- target_recovery_comparison.csv contains one percent-gain value per single-deaminase configuration relative to the dual-editor construct
- Valid editing sites meet the minimum coverage threshold of 10 before Poisson filtering

## Limitations

- Sensitivity comparisons assume the published reference target sets (Extended Data Fig. 2b,c; Supplementary Table 3) are complete and unbiased; recovery is bounded by the reference set's quality
- The percent-gain estimate is configuration-dependent: changing binSize, coverage, or Poisson thresholds alters the comparison and may fall outside the 20–30% range
- The benchmark was established in the RPE cell / RBP dataset context (RBP_Edit_GE_RPE_DATA.tsv); transfer to other cell types or tissues is not directly validated

## Evidence

- [supplementary] Dual-editor sensitivity gain over single editors: [evidence span withheld — anchor status: not_found]
