---
name: two-round-unique-genome-mapping
description: Use when processing paired-end FASTQ from MAPIT-seq (or similar edited-base
  RBP-mapping assays) and you need rRNA/tRNA-depleted, uniquely aligned genomic BAMs
  before GATK-based editing calling.
when_to_use_negative:
- Input is already an aligned BAM from a different pipeline with no FASTQ available
  — abundant-RNA filtering and two-round unique mapping cannot be re-run
- The analysis goal is variant calling on genomic DNA, where rRNA/tRNA filtering and
  FR strandness assumptions from RNA assays do not apply
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: BWA-MEM
  role: Indexes the abundant RNA sequences (rRNA, tRNA, mt-rRNA, mt-tRNA) used to
    filter contaminating reads before genome alignment
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: HISAT2
  role: Genome aligner used in the MAPIT-seq mapping stage for the two-round unique
    alignment against GRCh38
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: conda
  role: Installation channel for all pipeline software ('conda install snakemake -c
    bioconda -c conda-forge')
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: Snakemake
  role: Workflow manager recommended for installing and driving the MAPIT-seq pipeline
  repo: https://github.com/WangLabPKU/MAPIT-seq
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:548fb1f117d9fe1da046ba2cb59e70a26e23db91ea03141828eaba155e7d6702
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
  - Snakemake
  - conda
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# Two-round unique genome mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MAPIT-seq read-alignment stage that first filters abundant RNAs (rRNA, tRNA, mt-rRNA, mt-tRNA) indexed with BWA-MEM, then performs two rounds of uniquely-mapped alignment of paired-end FASTQ reads against the GRCh38 genome to produce clean BAMs for downstream RNA-editing detection.

## When to use

Use when processing paired-end FASTQ from MAPIT-seq (or similar edited-base RBP-mapping assays) and you need rRNA/tRNA-depleted, uniquely aligned genomic BAMs before GATK-based editing calling.

## When NOT to use

- Input is already an aligned BAM from a different pipeline with no FASTQ available — abundant-RNA filtering and two-round unique mapping cannot be re-run
- The analysis goal is variant calling on genomic DNA, where rRNA/tRNA filtering and FR strandness assumptions from RNA assays do not apply

## Inputs

- Paired-end FASTQ files (R1.fq.gz, R2.fq.gz) from MAPIT-seq libraries
- GRCh38 reference genome FASTA with 'chr'-prefixed chromosome names
- GFF3 genome annotation
- Abundant RNA sequences (rRNA, tRNA, mt-rRNA, mt-tRNA) indexed by BWA-MEM

## Outputs

- Uniquely-mapped, strand-aware (FR) BAM alignments per sample/replicate under the Mapit_result directory
- Fine-tuned alignments from 'Mapit finetuning' ready for GATK editing calling

## How to apply

Prepare a 'chr'-prefixed GRCh38 reference FASTA plus GFF3 annotation, and index the abundant RNA sequences (rRNA, tRNA, mt-rRNA, mt-tRNA, provided in the `ref` fold) with BWA-MEM. Run 'Mapit mapping' with the paired FASTQs and --rna-strandness FR so the first pass removes reads matching abundant RNA via BWA-MEM and the remaining reads undergo two-round uniquely mapping to the genome; specify sample (-n) and replicate (-r) names so outputs land under the Mapit_result directory. Follow with 'Mapit finetuning -v GRCh38' to fine-tune the alignments, since the uniquely-mapped, stranded BAMs feed directly into the 'callediting' (GATK RNA editing detection) step. Judge success by clean alignments that lack abundant-RNA contamination, retain strand information (FR), and are consistent with the sample/replicate naming used in later editing-count tables.

## Related tools

- **BWA-MEM** (Indexes the abundant RNA sequences (rRNA, tRNA, mt-rRNA, mt-tRNA) used to filter contaminating reads before genome alignment) — https://github.com/WangLabPKU/MAPIT-seq
- **HISAT2** (Genome aligner used in the MAPIT-seq mapping stage for the two-round unique alignment against GRCh38) — https://github.com/WangLabPKU/MAPIT-seq
- **conda** (Installation channel for all pipeline software ('conda install snakemake -c bioconda -c conda-forge')) — https://github.com/WangLabPKU/MAPIT-seq
- **Snakemake** (Workflow manager recommended for installing and driving the MAPIT-seq pipeline) — https://github.com/WangLabPKU/MAPIT-seq

## Examples

```
Mapit mapping -v GRCh38 --fq R1.fq.gz --fq2 R2.fq.gz --rna-strandness FR -n <sample> -r <rep> -o Mapit_result -t <threads>
```

## Evaluation signals

- Mapit_result output directory exists and contains alignments named consistently with the -n <sample> and -r <rep> arguments
- Downstream 'Mapit callediting' produces 6-Edit_calling records, confirming alignments were acceptable to GATK
- Abundant RNA (rRNA, tRNA, mt-rRNA, mt-tRNA) reads are removed — no abundant-RNA contamination should appear in the uniquely mapped output
- Alignments retain FR strand information (--rna-strandness FR) required by the editing-calling stage

## Limitations

- Requires 'chr'-prefixed reference names and per-chromosome split known-SNP datasets (dbSNP, 1000 Genomes, EVS/EVA) to be prepared beforehand
- Only uniquely mapped reads are retained; multi-mapping reads are discarded, which may lose signal in repetitive regions
- Assumes paired-end, stranded RNA libraries (rna-strandness FR); single-end or unstranded libraries are outside the demonstrated workflow

## Evidence

- [other] Run 'Mapit mapping -v GRCh38 --fq R1.fq.gz --fq2 R2.fq.gz --rna-strandness FR -n <sample> -r <rep> -o Mapit_result -t <threads>' to perform abundant RNA (rRNA, tRNA, mt-rRNA, mt-tRNA, indexed by BWA-MEM) filtering plus two-round unique alignment to the genome.: [evidence span withheld — anchor status: not_found]
- [readme] All softwares in our pipeline are available in conda. We recommend you to install these softwares via conda: [evidence span withheld — anchor status: not_found]
