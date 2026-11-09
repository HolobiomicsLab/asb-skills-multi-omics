---
name: umi-extraction-into-read-names
description: Use when processing sci-L3 FASTQ reads that must be labeled with their
  SSS barcode, IVT UMI, and three-level combinatorial barcode (Tn5/RT and ligation)
  before alignment or single-cell demultiplexing.
when_to_use_negative:
- Input is already aligned BAM data — use sss_bam2.py for post-alignment single-cell
  splitting instead of re-extracting barcodes from reads
- Reads do not originate from a three-round sci-L3 combinatorial indexing scheme (no
  RT primer, SSS barcode, or Tn5/ligation barcode structure)
- Only adapter trimming is needed without barcode/UMI annotation — cutadapt alone
  suffices
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: fastq2.py
  role: Extracts and appends SSS barcode and IVT UMI sequences to read names and generates
    sample. FASTQ indexing from the fastq directory
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: trimmed2.py
  role: Generates the trimmed. file that indexes the annotated trimmed FASTQ files
    per sample for downstream pipeline stages
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: cutadapt
  role: Trims the Tn5 MEDS adapter sequence from reads prior to barcode matching
- name: Snakemake
  role: Orchestrates the read-processing workflow via configs/config.yaml
  repo: https://github.com/recombinationlab/sciL3Pipe
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:169de222b8a0c88b80727d10d92de729261c80c5dee6a52668be92c96b782249
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/recombinationlab/sciL3Pipe
  repo_url: https://github.com/recombinationlab/sciL3Pipe
  tools:
  - Python
  - Snakemake
  - cutadapt
  - fastq2.py
  - trimmed2.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# UMI extraction into read names

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

In the sciL3Pipe read-processing stage, the SSS barcode and IVT UMI sequences are extracted from oriented read pairs and appended to the read names, alongside combinatorial first-round (8-nt Tn5/RT) and second-round (7-nt ligation) barcode matching, so every read carries its single-cell barcode identity into downstream alignment.

## When to use

Use when processing sci-L3 FASTQ reads that must be labeled with their SSS barcode, IVT UMI, and three-level combinatorial barcode (Tn5/RT and ligation) before alignment or single-cell demultiplexing.

## When NOT to use

- Input is already aligned BAM data — use sss_bam2.py for post-alignment single-cell splitting instead of re-extracting barcodes from reads
- Reads do not originate from a three-round sci-L3 combinatorial indexing scheme (no RT primer, SSS barcode, or Tn5/ligation barcode structure)
- Only adapter trimming is needed without barcode/UMI annotation — cutadapt alone suffices

## Inputs

- Raw paired-end FASTQ files (R1/R2) from sci-L3 libraries, indexed via sample. produced by fastq2.py
- config.yaml with rt_primer, rt_mismatches, sss_mismatches, tn5_mismatches, ligation_mismatches
- Barcode resource files: barcode_tn5.txt, barcode_ligation_104.txt, SSS barcode file

## Outputs

- Annotated trimmed FASTQ files with SSS barcode and IVT UMI appended to read names and combinatorial barcode labels
- RNA-versus-DNA tag per read (rt_UMI versus GGGGGG signature)
- trimmed. index file mapping annotated trimmed FASTQ files per sample

## How to apply

First parse the config.yaml parameters (rt_primer GGGATGCAGCTCGCTCCTG with rt_mismatches 3, sss_mismatches 0, tn5_mismatches 1, ligation_mismatches 1) and the barcode resource files barcode_tn5.txt, barcode_ligation_104.txt, and the SSS barcode file. Orient each read pair by searching for the RT primer and third-round barcode allowing up to 3 mismatches, then extract the SSS barcode and IVT UMI sequences and append them to the read names. Trim the Tn5 MEDS adapter with cutadapt, then match the first-round 8-nt Tn5 barcode and the second-round 7-nt ligation barcode against the provided lists, each allowing 1 mismatch (Levenshtein distance). Emit the RNA-versus-DNA tag by detecting the rt_UMI versus GGGGGG signature and write annotated trimmed FASTQ files; run trimmed2.py to generate the trimmed. index file per sample for downstream stages.

## Related tools

- **fastq2.py** (Extracts and appends SSS barcode and IVT UMI sequences to read names and generates sample. FASTQ indexing from the fastq directory) — https://github.com/recombinationlab/sciL3Pipe
- **trimmed2.py** (Generates the trimmed. file that indexes the annotated trimmed FASTQ files per sample for downstream pipeline stages) — https://github.com/recombinationlab/sciL3Pipe
- **cutadapt** (Trims the Tn5 MEDS adapter sequence from reads prior to barcode matching)
- **Snakemake** (Orchestrates the read-processing workflow via configs/config.yaml) — https://github.com/recombinationlab/sciL3Pipe

## Examples

```
python fastq2.py --fastq_dir tests/fastqs/
```

## Evaluation signals

- Every annotated read name contains the extracted SSS barcode and IVT UMI sequences
- First-round Tn5 barcodes (8 nt) and second-round ligation barcodes (7 nt) match a barcode-list entry within 1 mismatch (Levenshtein distance)
- Read-pair orientation succeeds only when the RT primer GGGATGCAGCTCGCTCCTG is found within rt_mismatches 3 and the SSS barcode within sss_mismatches 0
- The RNA-versus-DNA tag is consistent with detection of the rt_UMI versus GGGGGG signature
- trimmed2.py produces a trimmed. index file covering all annotated trimmed FASTQ files per sample

## Limitations

- Allowing 1 mismatch in barcode matching (Tn5, ligation) can conflate barcodes differing by a single nucleotide; the SSS barcode allows 0 mismatches
- Reads failing RT-primer orientation (allowed 3 mismatches) or barcode matching are lost, contributing to reduced recovered single cells with additional barcoding rounds
- Low recovery of RNA reads per cell in the sci-L3-RNA/ATAC co-assay limits the number of reads carrying usable rt_UMI-based RNA tags

## Evidence

- [methods] The pipeline identifies first-round barcodes (Tn5 or RT, 8 nt, 1 mismatch allowed by Levenshtein distance) and second-round barcodes (ligation, 7 nt, 1 mismatch allowed: [evidence span withheld — anchor status: not_found]
- [methods] Orient each read pair by searching for the RT primer and third-round barcode allowing up to 3 mismatches, then extract the SSS barcode and IVT UMI sequences: [evidence span withheld — anchor status: not_found]
- [methods] python fastq2.py --fastq_dir tests/fastqs/: [evidence span withheld — anchor status: not_found]
- [other] python trimmed2.py --fastq_dir tests/trimmed_fastq/: [evidence span withheld — anchor status: not_found]
