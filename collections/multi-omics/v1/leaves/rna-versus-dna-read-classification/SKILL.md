---
name: rna-versus-dna-read-classification
description: Use when processing sci-L3 paired-end FASTQ reads from the joint RNA/chromatin-accessibility
  (RNA/ATAC) co-assay, where reads must be labeled RNA or DNA so downstream splitting
  can separate transcriptome from accessibility data per single cell.
when_to_use_negative:
- Data is from a DNA-only sci-L3 modality (sci-L3-WGS, sci-L3-Strand-seq, sci-L3-Hi-C)
  where no RNA readout exists and every read is DNA-derived
- Reads are already aligned BAMs — classification must happen at the FASTQ stage before
  cutadapt trimming and bwa/bowtie2 alignment (use sss_bam2.py for post-alignment
  splitting instead)
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: fastq2.py
  role: Read-processing script that orients reads, extracts barcodes/UMIs, and emits
    the RNA-versus-DNA tag into annotated trimmed FASTQs
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: cutadapt
  role: Trims the Tn5 MEDS adapter sequence from reads prior to writing the annotated
    trimmed FASTQ files
- name: trimmed2.py
  role: Generates the trimmed. index of annotated trimmed FASTQ files per sample so
    classified reads can be consumed downstream
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: Snakemake
  role: Orchestrates the read-processing workflow defined in configs/config.yaml
  repo: https://github.com/recombinationlab/sciL3Pipe
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:f09572e71e273a2a92880f511a096332d520e22c7c6fc30ef08a42444d08e36d
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
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# rna-versus-dna-read-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies each read in sci-L3 RNA/ATAC co-assay data as RNA-derived or DNA-derived during barcode extraction, using the presence of the rt_UMI versus a GGGGGG signature in the read, and appends this tag to the annotated trimmed FASTQ output.

## When to use

Use when processing sci-L3 paired-end FASTQ reads from the joint RNA/chromatin-accessibility (RNA/ATAC) co-assay, where reads must be labeled RNA or DNA so downstream splitting can separate transcriptome from accessibility data per single cell.

## When NOT to use

- Data is from a DNA-only sci-L3 modality (sci-L3-WGS, sci-L3-Strand-seq, sci-L3-Hi-C) where no RNA readout exists and every read is DNA-derived
- Reads are already aligned BAMs — classification must happen at the FASTQ stage before cutadapt trimming and bwa/bowtie2 alignment (use sss_bam2.py for post-alignment splitting instead)

## Inputs

- Paired-end raw FASTQ files from sci-L3 RNA/ATAC libraries (organized per sample via sample. from fastq2.py)
- config.yaml with rt_primer GGGATGCAGCTCGCTCCTG, rt_mismatches 3, sss_mismatches 0, tn5_mismatches 1, ligation_mismatches 1
- Barcode resource files: barcode_tn5.txt, barcode_ligation_104.txt, and the SSS barcode file

## Outputs

- Annotated, adapter-trimmed FASTQ files with RNA-versus-DNA tags, SSS barcodes, and IVT UMIs appended to read names
- trimmed. index file mapping annotated trimmed FASTQs per sample for downstream pipeline stages

## How to apply

During read orientation and barcode extraction (fastq2.py step of sciL3Pipe), first locate the RT primer GGGATGCAGCTCGCTCCTG allowing up to 3 mismatches (rt_mismatches: 3), then extract the SSS barcode (sss_mismatches: 0) and IVT UMI and append them to read names. The RNA-versus-DNA call is emitted by detecting the rt_UMI versus a GGGGGG signature: reads carrying the rt_UMI sequence are RNA-derived, whereas reads showing the GGGGGG signature are DNA-derived. Write the classification tag into the annotated, adapter-trimmed (cutadapt, Tn5 MEDS adapter) FASTQ files, then index them per sample with trimmed2.py via the trimmed. file for downstream stages. Because this runs before alignment, the classification relies solely on sequence signatures in the raw reads, not on mapping to the genome.

## Related tools

- **fastq2.py** (Read-processing script that orients reads, extracts barcodes/UMIs, and emits the RNA-versus-DNA tag into annotated trimmed FASTQs) — https://github.com/recombinationlab/sciL3Pipe
- **cutadapt** (Trims the Tn5 MEDS adapter sequence from reads prior to writing the annotated trimmed FASTQ files)
- **trimmed2.py** (Generates the trimmed. index of annotated trimmed FASTQ files per sample so classified reads can be consumed downstream) — https://github.com/recombinationlab/sciL3Pipe
- **Snakemake** (Orchestrates the read-processing workflow defined in configs/config.yaml) — https://github.com/recombinationlab/sciL3Pipe

## Examples

```
python fastq2.py --fastq_dir tests/fastqs/; snakemake --configfile configs/config.yaml
```

## Evaluation signals

- Every read name in the annotated trimmed FASTQ carries a well-formed SSS barcode, IVT UMI, and an explicit RNA or DNA tag
- Reads failing orientation against the RT primer GGGATGCAGCTCGCTCCTG (within 3 mismatches) or the third-round barcode are excluded rather than mislabeled
- First-round Tn5 (8 nt) and second-round ligation (7 nt) barcodes each match their barcode lists (barcode_tn5.txt, barcode_ligation_104.txt) within 1 mismatch
- trimmed2.py produces a trimmed. file that indexes exactly the annotated trimmed FASTQs emitted per sample, with counts consistent with the input sample.
- Downstream, RNA-tagged reads should map predominantly as transcriptome data and DNA-tagged reads as accessibility/WGS data when split per cell

## Limitations

- Classification depends on the rt_UMI versus GGGGGG sequence signature before alignment, so reads with primer-region errors beyond the allowed mismatch budget may be lost or misclassified
- The sci-L3-RNA/ATAC co-assay has low recovery of RNA reads per cell, limiting the yield of RNA-classified reads
- A speculated (not experimentally verified) failure mechanism: RT enzymes may use DNA as template and generate double-stranded cDNA inserted single-endedly by Tn5, preventing further amplification and biasing RNA recovery

## Evidence

- [methods] workflow step 5: [evidence span withheld — anchor status: not_found]
- [methods] config parameters: [evidence span withheld — anchor status: not_found]
- [readme] trimmed2.py entry point: [evidence span withheld — anchor status: not_found]
