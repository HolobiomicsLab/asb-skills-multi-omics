---
name: abundant-rna-read-filtering-alignment
description: Use when starting from MAPIT-seq paired-end FASTQ files (R1/R2) and you
  need to strip rRNA/tRNA/mt-rRNA/mt-tRNA reads and obtain uniquely aligned genomic
  reads before RNA editing detection.
when_to_use_negative:
- Editing events are already called and the task is differential editing analysis
  or RBP target calling (use calltargets instead)
- Reads are single-end or strandness is unknown, since the mapping step expects paired
  FASTQs with --rna-strandness FR/FR-FIRSTSTRAND-style specification
- The experiment is not MAPIT-seq/hADAR2dd–rAPOBEC1 editing-based; generic RNA-seq
  quantification pipelines may be more appropriate
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: BWA-MEM
  role: Indexes the abundant RNA sequences (rRNA, tRNA, mt-rRNA, mt-tRNA) used to
    filter reads before genomic alignment
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: MAPIT-seq 'Mapit mapping'
  role: Wraps abundant RNA filtering plus two-round unique alignment of paired-end
    reads to GRCh38
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: conda
  role: Recommended installation channel for all pipeline software
  repo: https://github.com/WangLabPKU/MAPIT-seq
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:b63ae9becc2ecea6d346a6007b7a8f0e9954cac9df9d80039348a93c92bd3023
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
  - HISAT2
  - MAPIT-seq 'Mapit mapping'
  - conda
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# abundant-rna-read-filtering-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

The MAPIT-seq mapping stage: filter reads against abundant RNAs (rRNA, tRNA, mt-rRNA, mt-tRNA) via a BWA-MEM index, then perform two-round unique alignment of the remaining reads to the GRCh38 genome, producing uniquely mapped BAMs that feed the finetuning and editing-calling stages.

## When to use

Use when starting from MAPIT-seq paired-end FASTQ files (R1/R2) and you need to strip rRNA/tRNA/mt-rRNA/mt-tRNA reads and obtain uniquely aligned genomic reads before RNA editing detection.

## When NOT to use

- Editing events are already called and the task is differential editing analysis or RBP target calling (use calltargets instead)
- Reads are single-end or strandness is unknown, since the mapping step expects paired FASTQs with --rna-strandness FR/FR-FIRSTSTRAND-style specification
- The experiment is not MAPIT-seq/hADAR2dd–rAPOBEC1 editing-based; generic RNA-seq quantification pipelines may be more appropriate

## Inputs

- Paired-end FASTQ files (R1.fq.gz, R2.fq.gz) from MAPIT-seq libraries
- GRCh38 reference genome FASTA with 'chr'-prefixed chromosome names
- GFF3 gene annotation (GENCODE GRCh38 release 40)
- RepeatMasker annotation (rmsk.txt.gz)
- Abundant RNA sequences (rRNA, tRNA, mt-rRNA, mt-tRNA) provided in the 'ref' folder

## Outputs

- Uniquely mapped, strand-aware BAM alignments per sample/replicate under the Mapit_result output directory
- Filtered-out abundant RNA read counts (implicit in mapping reports)

## How to apply

First build BWA-MEM indexes of the abundant RNA sequences (rRNA, tRNA, mt-rRNA, mt-tRNA, provided in the 'ref' folder) alongside the GRCh38 reference genome FASTA, GFF3 annotation, and RepeatMasker annotation with 'chr'-prefixed chromosome names. Run 'Mapit mapping' with --fq/--fq2 for the paired FASTQs, --rna-strandness FR to encode strand information, and -n/-r to tag sample and replicate; the command first filters reads matching the abundant RNA index and then performs two-round uniquely mapping against the genome, retaining only uniquely placed reads. Because MAPIT-seq simultaneously profiles the transcriptome and RBP–RNA interactions, preserving strandness during alignment matters for downstream differential editing (calltargets) analysis. After mapping, alignments are refined by the separate 'Mapit finetuning' step, so this stage should output clean, uniquely mapped alignments under the Mapit_result directory rather than editing calls. Verify success by checking that alignments exist per sample/replicate and that downstream callediting receives the full sample list.

## Related tools

- **BWA-MEM** (Indexes the abundant RNA sequences (rRNA, tRNA, mt-rRNA, mt-tRNA) used to filter reads before genomic alignment) — https://github.com/WangLabPKU/MAPIT-seq
- **MAPIT-seq 'Mapit mapping'** (Wraps abundant RNA filtering plus two-round unique alignment of paired-end reads to GRCh38) — https://github.com/WangLabPKU/MAPIT-seq
- **conda** (Recommended installation channel for all pipeline software) — https://github.com/WangLabPKU/MAPIT-seq

## Examples

```
Mapit mapping -v GRCh38 --fq R1.fq.gz --fq2 R2.fq.gz --rna-strandness FR -n sample1 -r rep1 -o Mapit_result -t 8
```

## Evaluation signals

- Alignment output directory (Mapit_result) contains BAMs for every -n/-r sample/replicate combination supplied
- Residual rRNA/tRNA/mt-rRNA/mt-tRNA content in the aligned BAMs is minimal relative to the abundant RNA index hits
- Only uniquely mapped reads are retained (two-round uniquely mapping), and strandness matches the --rna-strandness FR setting
- Downstream 'Mapit finetuning' runs without missing-alignment errors, confirming the mapping stage produced complete inputs
- Reference assets (genome FASTA, GFF3, RepeatMasker, SNP sets) use consistent 'chr'-prefixed chromosome naming

## Limitations

- Requires pre-split, 'chr'-prefixed reference and SNP assets; mismatched chromosome nomenclature breaks alignment/annotation consistency
- Only uniquely mapped reads survive the two-round scheme, so multi-mapping reads (e.g., from repetitive regions) are discarded before editing calling
- Strandness must be specified correctly (FR in the documented workflow); incorrect settings propagate into strand-aware editing detection

## Evidence

- [methods] The first step combines abundant RNA filtering and read alignment: "The first step combines abundant RNA filtering and read alignment"
- [methods] BWA-MEM index of abundant RNA: "Abundant RNA (rRNA, tRNA, mt-rRNA and mt-tRNA) sequence (provided in `ref` fold) and create the index by **BWA-MEM**."
- [other] Mapit mapping command from source card workflow: [evidence span withheld — anchor status: not_found]
- [readme] MAPIT-seq simultaneity rationale: "This approach allows for the **simultaneous profiling** of **RBP–RNA interactions** and the **transcriptome** in tissues and single cells."
