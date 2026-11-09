---
name: rbp-target-identification
description: Use when calling RBP target transcripts from per-sample MAPIT-seq RNA
  editing event tables (e.g. *_Edit_*_DATA.tsv), comparing a deaminase-expressing
  treatment against its control to recover bound RNAs.
when_to_use_negative:
- Input is raw sequencing data or alignments rather than callediting-derived editing
  event tables - run the mapping, finetuning and callediting steps (GATK RNA editing
  detection) first
- The experiment lacks dual/single-deaminase editing signal (e.g. no hADAR2dd or rAPOBEC1
  enzyme comparison against a control), since target calling depends on differential
  editing enrichment
- Goal is transcriptome quantification only (gene expression co-profiling) rather
  than identifying RBP-bound transcripts
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: MAPIT-seq
  role: Provides the calltargets step (differential editing analysis to call RBP binding
    targets) and the editing event tables it consumes
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: GATK
  role: Upstream callediting step that generates the per-sample RNA editing event
    tables used for target calling
- name: SAILOR
  role: Alternative editing-site finding tool fed by the 'prepare' step of the MAPIT-seq
    workflow
  repo: https://github.com/YeoLab/FLARE
- name: REDItools2
  role: Parallel RNA editing event detection tool referenced in the MAPIT-seq ecosystem
    for producing editing event tables
  repo: https://github.com/BioinfoUNIBA/REDItools2
- name: conda
  role: Installation channel for all MAPIT-seq pipeline software including the calltargets
    tooling
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:6c196889df4ea08155eff1ccae498de0c38c6c9eb3dabad2e3fd55801c1c1a0a
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
  - SAILOR
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# rbp-target-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify RNA-binding protein (RBP) target transcripts from MAPIT-seq data by running the 'Mapit calltargets' differential editing analysis on adjacent RNA editing events introduced by hADAR2dd and rAPOBEC1, calling targets with editing fold enrichment > 2, MAPIT score > 0.5 and adjusted P < 0.05.

## When to use

Use when calling RBP target transcripts from per-sample MAPIT-seq RNA editing event tables (e.g. *_Edit_*_DATA.tsv), comparing a deaminase-expressing treatment against its control to recover bound RNAs.

## When NOT to use

- Input is raw sequencing data or alignments rather than callediting-derived editing event tables - run the mapping, finetuning and callediting steps (GATK RNA editing detection) first
- The experiment lacks dual/single-deaminase editing signal (e.g. no hADAR2dd or rAPOBEC1 enzyme comparison against a control), since target calling depends on differential editing enrichment
- Goal is transcriptome quantification only (gene expression co-profiling) rather than identifying RBP-bound transcripts

## Inputs

- Per-sample RNA editing event table produced by the callediting step (e.g. RBP_Edit_GE_RPE_DATA.tsv or *_Edit_*_DATA.tsv under 6-Edit_calling)
- Reference genome version annotation (GRCh38)
- Treatment and control sample names for each RBP (antibody-targeted vs control samples)
- Optional: published RBP target sets (e.g. Extended Data Fig. 2b,c, Supplementary Table 3) for validation of recovery rates

## Outputs

- Significant RBP target transcript/gene lists per component or construct configuration
- Adjusted P value table per target (e.g. prc2_target_adjusted_pvalues.csv)
- Target recovery comparison summary (e.g. target_recovery_comparison.csv)
- Input for downstream high-confidence edit cluster identification and de novo RBP-binding motif discovery

## How to apply

Run the MAPIT-seq 'calltargets' step (differential editing analysis) for each RBP treatment versus its control, setting --treatName/--controlName and specifying the genome version (e.g. -v GRCh38). Transcripts are split into 50-nt bins (binSize 50) and editing sites must pass a coverage filter (minimum coverage 10) and a Poisson filter on editing-site p-values (default 0.1, tightened to 0.05 for report-level significance). Targets are called using the MAPIT-seq thresholds: editing fold enrichment > 2, MAPIT score > 0.5, and adjusted P < 0.05 at gene/transcript level. The rationale is that editing events cluster on bound transcripts near the antibody-tethered deaminase, so differential editing enrichment between enzyme-expressing and control samples marks direct RBP-RNA interactions; the dual-editor fusion rAPOBEC1-hADAR2dd recovers approximately 20-30% more published targets than either single editor alone.

## Related tools

- **MAPIT-seq** (Provides the calltargets step (differential editing analysis to call RBP binding targets) and the editing event tables it consumes) — https://github.com/WangLabPKU/MAPIT-seq
- **GATK** (Upstream callediting step that generates the per-sample RNA editing event tables used for target calling)
- **SAILOR** (Alternative editing-site finding tool fed by the 'prepare' step of the MAPIT-seq workflow) — https://github.com/YeoLab/FLARE
- **REDItools2** (Parallel RNA editing event detection tool referenced in the MAPIT-seq ecosystem for producing editing event tables) — https://github.com/BioinfoUNIBA/REDItools2
- **conda** (Installation channel for all MAPIT-seq pipeline software including the calltargets tooling)

## Examples

```
Mapit calltargets -v GRCh38 --treatName <RBP_treatment> --controlName <control> --binSize 50 --coverage 10 -p 0.05
```

## Evaluation signals

- Reproduced XIST adjusted P values for EED, EZH2 and SUZ12 match reported 0.0463, 1.72e-3 and 9.94e-6 within numerical tolerance
- PRC2 core component target sets remain near-empty (XIST the only common target), consistent with weak PRC2-RNA binding
- Dual-editor rAPOBEC1-hADAR2dd configuration recovers approximately 20-30% more published targets than single-deaminase configurations
- All called editing sites satisfy minimum coverage of 10 and the Poisson filter threshold on editing-site p-values
- Called targets pass the target-calling thresholds: editing fold enrichment > 2, MAPIT score > 0.5, adjusted P < 0.05

## Limitations

- Core PRC2 components yield near-empty target sets, reflecting weak PRC2-RNA binding, so sensitivity is limited for low-binding RBPs
- The Poisson filter default for editing sites is p-value 0.1; report-level significance required tightening to 0.05, so default parameters alone may overcall sites
- Target recovery rates depend on the quality of the published target sets used for validation

## Evidence

- [results] sensitivity gain finding: [evidence span withheld — anchor status: not_found]
