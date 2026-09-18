---
name: rna-editing-event-detection-gatk
description: Use when you have MAPIT-seq BAM alignments (post-mapping and finetuning)
  with hADAR2dd and rAPOBEC1 deamination and need to call adjacent A-to-G and C-to-U
  editing events to mark RBP-bound transcripts.
when_to_use_negative:
- The goal is calling RBP binding targets by differential editing (the 'calltargets'
  stage) or preparing files for SAILOR rather than detecting editing events themselves
- Input reads lack the MAPIT-seq antibody-directed editing chemistry (hADAR2dd and
  rAPOBEC1 deamination), since ADAR/APOBEC edit signatures will not reflect RBP proximity
- Only single-end or non-uniquely mapped alignments exist, as the pipeline expects
  two-round uniquely mapping paired-end input
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: GATK
  role: Variant/editing calling engine invoked by the 'callediting' subcommand with
    '-e Both' to detect ADAR and APOBEC editing
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: BWA-MEM
  role: Indexes abundant RNA (rRNA, tRNA, mt-rRNA, mt-tRNA) sequences for the filtering
    step preceding editing detection
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: conda
  role: Recommended installation channel for all pipeline software including the editing-calling
    toolchain
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: SAILOR/FLARE
  role: Alternative/editing-site downstream pipeline supported by the 'prepare' step
    (prepare files for SAILOR) in the MAPIT-seq workflow
  repo: https://github.com/YeoLab/FLARE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:18935e70bd60cc6e49a48520c5ba240927cca28010c4acfe729c6539674b226f
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
  - BWA-MEM
  - GATK
  - HISAT2
  - SAILOR/FLARE
  - conda
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# rna-editing-event-detection-gatk

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MAPIT-seq's editing-detection stage: calling ADAR (A-to-G) and APOBEC (C-to-U) RNA editing events from two-round uniquely mapped alignments using GATK within the Mapit 'callediting' subcommand, producing per-sample editing count tables used to identify RBP-bound transcripts.

## When to use

Use when you have MAPIT-seq BAM alignments (post-mapping and finetuning) with hADAR2dd and rAPOBEC1 deamination and need to call adjacent A-to-G and C-to-U editing events to mark RBP-bound transcripts.

## When NOT to use

- The goal is calling RBP binding targets by differential editing (the 'calltargets' stage) or preparing files for SAILOR rather than detecting editing events themselves
- Input reads lack the MAPIT-seq antibody-directed editing chemistry (hADAR2dd and rAPOBEC1 deamination), since ADAR/APOBEC edit signatures will not reflect RBP proximity
- Only single-end or non-uniquely mapped alignments exist, as the pipeline expects two-round uniquely mapping paired-end input

## Inputs

- Paired FASTQ files (R1.fq.gz, R2.fq.gz) from MAPIT-seq libraries
- BAM alignments from 'Mapit mapping' and 'Mapit finetuning' (two-round uniquely mapped)
- GRCh38 reference genome FASTA and GFF3 annotation
- RepeatMasker (rmsk) annotation
- Known SNP VCFs/bedfiles split per chromosome (dbSNP, 1000 Genomes, EVS/EVA) with chr-prefixed names

## Outputs

- editing_counts_per_sample.tsv with one row per sample and separate A-to-G and C-to-U count columns
- Per-sample RNA editing event records under the 6-Edit_calling output directory
- Editing rates (A-to-G and C-to-U events per million bases) computable within 800-nt windows around PAR-CLIP peaks

## How to apply

After abundant RNA (rRNA, tRNA, mt-rRNA, mt-tRNA) filtering and two-round unique alignment to GRCh38, run 'Mapit callediting' with the GRCh38 reference, a comma-separated --sampleList, a --prefix (e.g. RBP), and '-e Both' so GATK detects both ADAR (A-to-G) and APOBEC (C-to-U) editing. Use known SNP datasets (dbSNP, 1000 Genomes, EVS/EVA) split per chromosome with 'chr'-prefixed names to exclude variants; candidate sites are filtered by coverage (default minimum 10) and a Poisson filter (default maximum p-value 0.1). Aggregate events per sample into separate A-to-G and C-to-U count columns and write editing_counts_per_sample.tsv; downstream, editing can be quantified as events per million bases within 800-nt windows flanking YTHDF2 PAR-CLIP peaks. Subsequent high-confidence edit cluster identification and differential editing analysis (calltargets) then pinpoint RBP-binding regions.

## Related tools

- **GATK** (Variant/editing calling engine invoked by the 'callediting' subcommand with '-e Both' to detect ADAR and APOBEC editing) — https://github.com/WangLabPKU/MAPIT-seq
- **BWA-MEM** (Indexes abundant RNA (rRNA, tRNA, mt-rRNA, mt-tRNA) sequences for the filtering step preceding editing detection) — https://github.com/WangLabPKU/MAPIT-seq
- **conda** (Recommended installation channel for all pipeline software including the editing-calling toolchain) — https://github.com/WangLabPKU/MAPIT-seq
- **SAILOR/FLARE** (Alternative/editing-site downstream pipeline supported by the 'prepare' step (prepare files for SAILOR) in the MAPIT-seq workflow) — https://github.com/YeoLab/FLARE

## Examples

```
Mapit callediting -v GRCh38 --sampleList sample1,sample2 --prefix RBP -e Both -t 8
```

## Evaluation signals

- editing_counts_per_sample.tsv exists, contains one row per sample listed in --sampleList, and has separate A-to-G and C-to-U count columns
- Per-sample count columns are consistent with the editing event records stored under the 6-Edit_calling output directory
- Editing events are reported for both ADAR (A-to-G) and APOBEC (C-to-U) classes when '-e Both' is used
- Candidate editing sites satisfy coverage >= 10 and Poisson filter p-value <= 0.1 (defaults)
- Normalized editing (events per million bases) computed within 800-nt windows flanking YTHDF2 PAR-CLIP peaks falls in a plausible range relative to background

## Limitations

- Editing quantification context (e.g. events per million bases within 800-nt windows flanking YTHDF2 PAR-CLIP peaks) is tied to peak-anchored windows, so results depend on available CLIP peak coordinates
- Known SNP resources must be split per chromosome and 'chr'-prefixed; chromosome nomenclature mismatches break the filtering
- Detection assumes editing introduced by hADAR2dd and rAPOBEC1 adjacent to the RBP binding site; unrelated deamination sources could confound calls

## Evidence

- [other] callediting command with -e Both flags: [evidence span withheld — anchor status: not_found]
- [results] editing output quantification context: [evidence span withheld — anchor status: not_found]
