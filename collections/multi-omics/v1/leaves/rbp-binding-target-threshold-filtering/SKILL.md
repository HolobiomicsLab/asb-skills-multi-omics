---
name: rbp-binding-target-threshold-filtering
description: Use when you have a MAPIT-seq differential editing statistics table (e.g.,
  RBP_Edit_GE_RPE_DATA.tsv from the calltargets step) and need to define the RBP's
  target gene set under default thresholds.
when_to_use_negative:
- Input is raw sequencing data or alignments that have not yet passed the callediting/calltargets
  steps — editing statistics do not exist yet to threshold.
- Goal is to call individual RNA editing sites rather than RBP binding targets — use
  the Poisson filter of editing sites and coverage filtering directly, not the target-level
  thresholds.
- Data come from methods without an editing signal (e.g., CLIP-seq peaks or expression-only
  scRNA-seq) — the fold enrichment/MAPIT score thresholds are undefined for those
  inputs.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GATK
  role: Underlying engine for the callediting step that produces the RNA editing calls
    subsequently filtered into RBP targets
- name: MAPIT-seq pipeline
  role: Provides the calltargets differential editing analysis and the target-calling
    threshold parameters (--pvalue, --coverage)
  repo: https://github.com/WangLabPKU/MAPIT-seq
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:181fb45839fc6c4d87b4f68bb16026fc5eb8a96113156938e20d66f68066130e
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
  - MAPIT-seq pipeline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# RBP binding target threshold filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts MAPIT-seq differential RNA-editing statistics into a called RBP binding target list by applying the pipeline's default target-calling thresholds (editing fold enrichment > 2, MAPIT score > 0.5, adjusted P < 0.05) to GATK-based editing calls, optionally followed by overlap testing against an orthogonal RBP binding dataset such as PAR-CLIP.

## When to use

Use when you have a MAPIT-seq differential editing statistics table (e.g., RBP_Edit_GE_RPE_DATA.tsv from the calltargets step) and need to define the RBP's target gene set under default thresholds.

## When NOT to use

- Input is raw sequencing data or alignments that have not yet passed the callediting/calltargets steps — editing statistics do not exist yet to threshold.
- Goal is to call individual RNA editing sites rather than RBP binding targets — use the Poisson filter of editing sites and coverage filtering directly, not the target-level thresholds.
- Data come from methods without an editing signal (e.g., CLIP-seq peaks or expression-only scRNA-seq) — the fold enrichment/MAPIT score thresholds are undefined for those inputs.

## Inputs

- MAPIT-seq differential editing statistics table from the calltargets step (e.g., RBP_Edit_GE_RPE_DATA.tsv under 6-Edit_calling/)
- GATK-called RNA editing site table with fold enrichment, MAPIT score, and adjusted P values
- Orthogonal RBP binding target list (e.g., published G3BP1 PAR-CLIP targets) for overlap analysis

## Outputs

- Thresholded RBP binding target list (genes/transcripts passing fold enrichment > 2, MAPIT score > 0.5, adjusted P < 0.05)
- Overlap statistics between MAPIT-seq targets and PAR-CLIP targets (e.g., MAPIT ∩ PAR-CLIP intersection count for Venn diagrams and comparison against Supplementary Table 2)

## How to apply

Run the MAPIT-seq pipeline's 'callediting' step (GATK-based RNA editing detection) and then the 'calltargets' step (differential editing analysis) to produce a differential editing statistics table per RBP. Filter candidate binding targets with the default thresholds: editing fold enrichment > 2, MAPIT score > 0.5, and adjusted P < 0.05. Retain only editing sites passing the Poisson significance filter (maximum p-value, e.g., 0.05 via --pvalue) and the minimum coverage of 10 for valid editing sites (--coverage). Optionally intersect the thresholded target list with an orthogonal target set (e.g., published G3BP1 PAR-CLIP targets) and assess overlap significance; for G3BP1 MAPIT-seq in HEK293T cells this yields 4,769 target genes detected by both methods, with MAPIT-seq preferentially recovering the higher-confidence PAR-CLIP targets.

## Related tools

- **GATK** (Underlying engine for the callediting step that produces the RNA editing calls subsequently filtered into RBP targets)
- **MAPIT-seq pipeline** (Provides the calltargets differential editing analysis and the target-calling threshold parameters (--pvalue, --coverage)) — https://github.com/WangLabPKU/MAPIT-seq

## Evaluation signals

- Every retained target has editing fold enrichment > 2, MAPIT score > 0.5, and adjusted P < 0.05.
- Editing sites underlying called targets pass the Poisson filter (p ≤ 0.05 in the example calltargets command) and minimum coverage of 10.
- The MAPIT ∩ PAR-CLIP intersection count matches the reported 4,769 target genes for G3BP1 MAPIT-seq in HEK293T cells (Fig. 2b,c, Supplementary Table 2).
- The overlap with PAR-CLIP targets reaches statistical significance, and higher-confidence PAR-CLIP targets are preferentially recovered.

## Limitations

- Default thresholds (fold enrichment > 2, MAPIT score > 0.5, adjusted P < 0.05) are experiment-specific defaults; the Poisson filter p-value defaults to 0.1 and coverage to 10, so the example's p = 0.05 is a stricter choice that must be stated explicitly.
- Target calling depends on upstream editing detection quality; the article notes the pipeline relies on filtering known SNPs (dbSNP, 1000 Genomes, EVS/EVA) and abundant RNAs, so contamination of editing calls will propagate into the target list.

## Evidence
