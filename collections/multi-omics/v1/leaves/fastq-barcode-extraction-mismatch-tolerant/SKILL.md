---
name: fastq-barcode-extraction-mismatch-tolerant
description: 'Use when sci-L3 combinatorial-indexing FASTQ reads must be demultiplexed
  into single-cell barcode identities: orienting reads on the RT primer GGGATGCAGCTCGCTCCTG
  and matching 8-nt Tn5 and 7-nt ligation barcodes with 1-mismatch tolerance.'
when_to_use_negative:
- Input is already aligned BAM output (e.g., SSS BAMs destined for sss_bam2.py / split_bam)
  rather than raw FASTQ needing barcode extraction
- Reads lack the sci-L3 three-round barcode architecture — no RT primer (GGGATGCAGCTCGCTCCTG)
  and no 8-nt Tn5 / 7-nt ligation whitelists apply, as in conventional single-index
  single-cell FASTQs
- The goal is downstream genome-scale analysis (bwa/bowtie2 alignment, Hi-C contact
  maps via docker-4dn-hic or juicer) rather than per-read barcode annotation
edam_operation: http://edamontology.org/operation_3238
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0160
tools:
- name: Snakemake
  role: Drives the read-processing workflow that performs barcode orientation, extraction,
    and annotation from raw FASTQ via --configfile configs/config.yaml
- name: fastq2.py
  role: Creates the sample. mapping of R1/R2 FASTQ files to samples so barcode annotation
    runs per sample (README shows the fastq2.py entry point)
  repo: github.com/recombinationlab/sciL3Pipe
- name: cutadapt
  role: Trims the Tn5 MEDS adapter sequence from reads before barcode matching and
    annotation
- name: Python
  role: Implements the Levenshtein-distance barcode matching, RT-primer orientation,
    and read-name annotation in the read-processing scripts
- name: trimmed2.py
  role: Generates the trimmed. index file that lists annotated trimmed FASTQs per
    sample for downstream pipeline stages
  repo: github.com/recombinationlab/sciL3Pipe
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:b574706226dcde414a6d5e173528d570056779b31b5f8ce70d9a96df3d91dd34
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/recombinationlab/sciL3Pipe
  tools:
  - Python
  - Snakemake
  - cutadapt
  - fastq2.py
  - trimmed2.py
  edam_operation: http://edamontology.org/operation_3238
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# fastq-barcode-extraction-mismatch-tolerant

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts and consolidates sci-L3's three-level combinatorial barcodes from raw paired-end FASTQ — first-round 8-nt Tn5, second-round 7-nt ligation, third-round, and SSS barcodes plus IVT UMI — using Levenshtein-distance-tolerant matching so every read is labeled with its single-cell barcode identity.

## When to use

Use when sci-L3 combinatorial-indexing FASTQ reads must be demultiplexed into single-cell barcode identities: orienting reads on the RT primer GGGATGCAGCTCGCTCCTG and matching 8-nt Tn5 and 7-nt ligation barcodes with 1-mismatch tolerance.

## When NOT to use

- Input is already aligned BAM output (e.g., SSS BAMs destined for sss_bam2.py / split_bam) rather than raw FASTQ needing barcode extraction
- Reads lack the sci-L3 three-round barcode architecture — no RT primer (GGGATGCAGCTCGCTCCTG) and no 8-nt Tn5 / 7-nt ligation whitelists apply, as in conventional single-index single-cell FASTQs
- The goal is downstream genome-scale analysis (bwa/bowtie2 alignment, Hi-C contact maps via docker-4dn-hic or juicer) rather than per-read barcode annotation

## Inputs

- Paired-end FASTQ files (R1/R2) from sci-L3 libraries, indexed by sample. generated from a fastq directory (e.g., tests/fastqs/)
- config.yaml run parameters: rt_primer GGGATGCAGCTCGCTCCTG, rt_mismatches 3, sss_mismatches 0, tn5_mismatches 1, ligation_mismatches 1
- Barcode whitelist files: barcode_tn5.txt (first-round 8-nt barcodes), barcode_ligation_104.txt (second-round 7-nt barcodes), and the SSS barcode file

## Outputs

- Annotated trimmed FASTQ files with the SSS barcode, IVT UMI, and first-/second-round barcodes appended to read names
- Per-read RNA-versus-DNA tags (rt_UMI vs GGGGGG signature)
- trimmed. index file mapping annotated trimmed FASTQ files per sample for downstream pipeline stages

## How to apply

Parse the config.yaml parameters — rt_primer GGGATGCAGCTCGCTCCTG with rt_mismatches 3, sss_mismatches 0, tn5_mismatches 1, ligation_mismatches 1 — together with the barcode resource files barcode_tn5.txt, barcode_ligation_104.txt, and the SSS barcode file. Orient each read pair by searching for the RT primer and third-round barcode allowing up to 3 mismatches, then extract the SSS barcode and IVT UMI sequences and append them to the read names. Trim the Tn5 MEDS adapter sequence with cutadapt, then match the first-round 8-nt Tn5 barcode and the second-round 7-nt ligation barcode against the whitelists, each allowing 1 mismatch by Levenshtein distance — the tight tolerance keeps assignments specific while correcting single-base sequencing errors. Emit the RNA-versus-DNA tag by detecting the rt_UMI versus GGGGGG signature, and write the annotated trimmed FASTQ files. Finally run trimmed2.py to generate the trimmed. file that indexes the annotated trimmed FASTQs per sample for downstream stages (split_bam: True; cutoff: 2000 reads per single cell, default 10000).

## Related tools

- **Snakemake** (Drives the read-processing workflow that performs barcode orientation, extraction, and annotation from raw FASTQ via --configfile configs/config.yaml)
- **fastq2.py** (Creates the sample. mapping of R1/R2 FASTQ files to samples so barcode annotation runs per sample (README shows the fastq2.py entry point)) — github.com/recombinationlab/sciL3Pipe
- **cutadapt** (Trims the Tn5 MEDS adapter sequence from reads before barcode matching and annotation)
- **Python** (Implements the Levenshtein-distance barcode matching, RT-primer orientation, and read-name annotation in the read-processing scripts)
- **trimmed2.py** (Generates the trimmed. index file that lists annotated trimmed FASTQs per sample for downstream pipeline stages) — github.com/recombinationlab/sciL3Pipe

## Examples

```
snakemake --configfile configs/config.yaml
```

## Evaluation signals

- Read names in the annotated trimmed FASTQs carry the extracted SSS barcode, IVT UMI, and consolidated 8-nt Tn5 + 7-nt ligation barcode, each whitelist match within the configured mismatch budget (tn5_mismatches 1, ligation_mismatches 1)
- Read orientation succeeds only for pairs whose RT primer/third-round barcode match within rt_mismatches 3; unmatchable pairs are excluded from the annotated output
- No Tn5 MEDS adapter sequence remains in the cutadapt-trimmed reads
- RNA-versus-DNA tags cleanly split reads into rt_UMI-bearing versus GGGGGG-signature classes per sample defined in sample.
- trimmed2.py's trimmed. index references exactly the set of annotated trimmed FASTQ files (e.g., in tests/trimmed_fastq/) with one entry per sample

## Limitations

- Barcode error correction is shallow: only 1 Levenshtein mismatch is tolerated for Tn5 and ligation barcodes and 0 for SSS barcodes, so barcode reads with more errors are discarded
- The 3-mismatch tolerance when searching the RT primer and third-round barcode trades orientation sensitivity for possible mis-assignment of reads
- Under the sci framework, additional rounds of cell barcoding reduce the number of recovered single cells expected by the number of cells sorted due to loss of per-cell coverage
- Low recovery of RNA reads per cell limits RNA barcode yield in the sci-L3-RNA/ATAC co-assay
- No software version numbers are stated for the pipeline stack (Snakemake, cutadapt, Python), complicating exact reproduction of barcode-matching behavior

## Evidence

- [methods] The pipeline matches first-round 8-nt Tn5 and second-round 7-nt ligation barcodes against the provided whitelists, each allowing 1 mismatch: [evidence span withheld — anchor status: not_found]
- [methods] Barcode-extraction parameters are set in config.yaml: rt_primer GGGATGCAGCTCGCTCCTG, rt_mismatches 3, sss_mismatches 0, tn5_mismatches 1, ligation_mismatches 1: [evidence span withheld — anchor status: not_found]
- [methods] trimmed2.py indexes the annotated trimmed FASTQ files per sample for downstream pipeline stages: [evidence span withheld — anchor status: not_found]
