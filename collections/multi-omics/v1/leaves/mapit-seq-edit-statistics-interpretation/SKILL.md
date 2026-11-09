---
name: mapit-seq-edit-statistics-interpretation
description: Use when a MAPIT-seq run has produced a differential editing statistics
  table and you need to threshold it into a target list, or when reproducing MAPIT
  ∩ PAR-CLIP overlap counts (e.g., 4,769 G3BP1 targets in HEK293T cells).
when_to_use_negative:
- Input is raw FASTQ or BAM alignments rather than an editing statistics table — use
  the upstream mapping/finetuning/callediting pipeline steps first
- The goal is detecting editing events themselves (edit site calling with REDItools2/SAILOR)
  rather than interpreting statistics to call RBP targets
- No orthogonal RBP reference set or supplementary expected counts exist and only
  site-level edit calling quality is needed
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GATK
  role: Engine underlying the MAPIT-seq callediting step for RNA editing detection
    whose statistics tables this skill interprets
- name: MAPIT-seq pipeline
  role: Provides the calltargets step (differential editing analysis to call RBP binding
    targets) with Poisson p-value (-p) and coverage (-c) filter options
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: Snakemake
  role: Workflow manager used to execute the MAPIT-seq pipeline steps that generate
    the editing statistics
  repo: https://github.com/WangLabPKU/MAPIT-seq
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:20bfb0ec540fd9e58097f578e09e0bccbfb387f4581577c2239073d4169c80ee
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
  - Snakemake
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# MAPIT-seq edit statistics interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets MAPIT-seq differential editing statistics (e.g., RBP_Edit_GE_RPE_DATA.tsv from the GATK-based callediting/calltargets steps) to call RBP binding targets using the paper's thresholds — editing fold enrichment > 2, MAPIT score > 0.5, adjusted P < 0.05 — plus Poisson (p ≤ 0.05) and coverage ≥ 10 editing-site filters, and to validate targets against orthogonal RBP reference sets such as PAR-CLIP.

## When to use

Use when a MAPIT-seq run has produced a differential editing statistics table and you need to threshold it into a target list, or when reproducing MAPIT ∩ PAR-CLIP overlap counts (e.g., 4,769 G3BP1 targets in HEK293T cells).

## When NOT to use

- Input is raw FASTQ or BAM alignments rather than an editing statistics table — use the upstream mapping/finetuning/callediting pipeline steps first
- The goal is detecting editing events themselves (edit site calling with REDItools2/SAILOR) rather than interpreting statistics to call RBP targets
- No orthogonal RBP reference set or supplementary expected counts exist and only site-level edit calling quality is needed

## Inputs

- MAPIT-seq differential editing statistics table (e.g., RBP_Edit_GE_RPE_DATA.tsv under 6-Edit_calling/) from the callediting (GATK RNA editing detection) step
- Editing site-level metrics: fold enrichment, MAPIT score, adjusted P value, Poisson filter p-value, per-site coverage
- Published PAR-CLIP target list for the same RBP and cell type (e.g., G3BP1 PAR-CLIP in HEK293T)

## Outputs

- Thresholded MAPIT-seq target gene list (fold enrichment > 2, MAPIT score > 0.5, adjusted P < 0.05)
- MAPIT ∩ PAR-CLIP intersection gene set and overlap count (e.g., 4,769 genes for G3BP1)
- Venn overlap visualization as in Fig. 2b,c, comparable to Supplementary Table 2

## How to apply

Load the GATK-derived differential editing table produced by the MAPIT-seq pipeline's callediting step (e.g., 6-Edit_calling/RBP_Edit_GE_RPE_DATA.tsv), then apply the calltargets default thresholds: editing fold enrichment > 2, MAPIT score > 0.5, and adjusted P < 0.05. Apply the Poisson significance filter (maximum p-value, set to 0.05 in the example calltargets command; tool default 0.1) and require minimum coverage of 10 for valid editing sites. The rationale is that editing events introduced by hADAR2dd and rAPOBEC1 adjacent to binding sites are the direct signal, so target calling is a differential editing analysis between RBP-edited and control conditions rather than peak calling. For validation, intersect the thresholded target gene list with a published PAR-CLIP target list (e.g., G3BP1 in HEK293T) and compare the MAPIT ∩ PAR-CLIP count against Supplementary Table 2 and the Fig. 2b,c Venn overlap; preferentially recovering higher-confidence PAR-CLIP targets indicates correct thresholding.

## Related tools

- **GATK** (Engine underlying the MAPIT-seq callediting step for RNA editing detection whose statistics tables this skill interprets)
- **MAPIT-seq pipeline** (Provides the calltargets step (differential editing analysis to call RBP binding targets) with Poisson p-value (-p) and coverage (-c) filter options) — https://github.com/WangLabPKU/MAPIT-seq
- **Snakemake** (Workflow manager used to execute the MAPIT-seq pipeline steps that generate the editing statistics) — https://github.com/WangLabPKU/MAPIT-seq

## Evaluation signals

- Default-threshold G3BP1 MAPIT-seq targets in HEK293T cells significantly overlap G3BP1 PAR-CLIP targets with 4,769 genes in the MAPIT ∩ PAR-CLIP intersection
- MAPIT-seq preferentially recovers higher-confidence PAR-CLIP targets — enrichment should increase with PAR-CLIP confidence tier
- Retained editing sites satisfy the Poisson filter (p-value ≤ 0.05 per the example command; tool default 0.1) and minimum coverage of 10
- Every called target passes all three default thresholds simultaneously: fold enrichment > 2, MAPIT score > 0.5, adjusted P < 0.05
- Intersection count matches Supplementary Table 2 and the Fig. 2b,c Venn rendering

## Limitations

- Edit site filtering defaults are looser than the example analysis (Poisson p-value default 0.1 vs 0.05 used; coverage default 10) — mismatched filter settings will change target counts
- Overlap significance depends on using the matching PAR-CLIP reference for the same RBP and cell type; G3BP1/HEK293T numbers do not transfer to other RBPs
- MAPIT-seq target calling is a differential editing analysis, so it inherits sensitivity to the callediting (GATK) step's SNP filtering (dbSNP, 1000 Genomes, EVS/EVA) and known-editing annotations

## Evidence

- [results] Default thresholds with 4,769 MAPIT ∩ PAR-CLIP genes: [evidence span withheld — anchor status: not_found]
- [results] Default target thresholds: [evidence span withheld — anchor status: not_found]
