---
name: tn5-meds-adapter-trimming
description: Use when raw sci-L3 FASTQ reads carry the Tn5 MEDS adapter sequence and
  must be trimmed after combinatorial barcode/UMI extraction before alignment.
when_to_use_negative:
- Input FASTQs are already trimmed and barcode-annotated by the fastq2.py stage —
  re-trimming would remove nothing and duplicate processing
- Data are not sci-L3 reads (e.g., standard Hi-C libraries processed via the 4dn docker-4dn-hic
  or juicer pipelines), which use different adapter and restriction-site handling
- The goal is post-alignment processing — sss_bam2.py on BWA alignment output, not
  read-level trimming
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: cutadapt
  role: Trims the Tn5 MEDS adapter sequence from reads during the read-processing
    stage
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: fastq2.py
  role: Upstream step that parses FASTQs, matches barcodes with configured mismatch
    tolerances, and writes the annotated trimmed FASTQ files
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: trimmed2.py
  role: Generates the trimmed. file that indexes the annotated trimmed FASTQ files
    per sample for downstream pipeline stages
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: Snakemake
  role: Orchestrates the read-processing workflow steps including adapter trimming
    via configfile-driven DAG execution
  repo: https://github.com/recombinationlab/sciL3Pipe
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:e68875fa534a2ab702a48707dd9b9337d5ee424648d331838bb20ed2c7f8f50d
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
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# tn5-meds-adapter-trimming

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trims the Tn5 MEDS adapter sequence from sci-L3 sequencing reads using cutadapt, within the sciL3Pipe read-processing stage that also extracts first-round (Tn5, 8 nt) and second-round (ligation, 7 nt) combinatorial barcodes before downstream alignment.

## When to use

Use when raw sci-L3 FASTQ reads carry the Tn5 MEDS adapter sequence and must be trimmed after combinatorial barcode/UMI extraction before alignment.

## When NOT to use

- Input FASTQs are already trimmed and barcode-annotated by the fastq2.py stage — re-trimming would remove nothing and duplicate processing
- Data are not sci-L3 reads (e.g., standard Hi-C libraries processed via the 4dn docker-4dn-hic or juicer pipelines), which use different adapter and restriction-site handling
- The goal is post-alignment processing — sss_bam2.py on BWA alignment output, not read-level trimming

## Inputs

- Raw paired-end FASTQ files (R1/R2) from sci-L3 sequencing, organized via sample. produced by fastq2.py
- config.yaml pipeline parameters (rt_primer, rt_mismatches 3, sss_mismatches 0, tn5_mismatches 1, ligation_mismatches 1)
- Barcode resource files: barcode_tn5.txt, barcode_ligation_104.txt, SSS barcode file

## Outputs

- Annotated, Tn5 MEDS adapter-trimmed FASTQ files with SSS barcode and IVT UMI appended to read names
- trimmed. index file (from trimmed2.py) mapping per-sample trimmed FASTQs for downstream stages

## How to apply

Within the sciL3Pipe read-processing stage, first parse config.yaml for parameters such as rt_primer GGGATGCAGCTCGCTCCTG, rt_mismatches 3, sss_mismatches 0, tn5_mismatches 1, and ligation_mismatches 1, plus the barcode resource files (barcode_tn5.txt, barcode_ligation_104.txt, SSS barcode file). Orient each read pair by searching for the RT primer and third-round barcode, then extract the SSS barcode and IVT UMI sequences and append them to read names. With barcodes annotated, trim the Tn5 MEDS adapter sequence from the reads using cutadapt, matching the 8-nt Tn5 and 7-nt ligation barcodes against the provided lists with 1 mismatch (Levenshtein distance) allowed. Emit the RNA-versus-DNA tag by detecting the rt_UMI versus GGGGGG signature and write the annotated, trimmed FASTQ files; downstream, trimmed2.py indexes these trimmed FASTQs per sample into the trimmed. file for subsequent pipeline stages. Correct application is judged by reads carrying appended barcode/UMI identifiers in their names and adapter-free sequence ready for bwa/bowtie2 alignment.

## Related tools

- **cutadapt** (Trims the Tn5 MEDS adapter sequence from reads during the read-processing stage) — https://github.com/recombinationlab/sciL3Pipe
- **fastq2.py** (Upstream step that parses FASTQs, matches barcodes with configured mismatch tolerances, and writes the annotated trimmed FASTQ files) — https://github.com/recombinationlab/sciL3Pipe
- **trimmed2.py** (Generates the trimmed. file that indexes the annotated trimmed FASTQ files per sample for downstream pipeline stages) — https://github.com/recombinationlab/sciL3Pipe
- **Snakemake** (Orchestrates the read-processing workflow steps including adapter trimming via configfile-driven DAG execution) — https://github.com/recombinationlab/sciL3Pipe

## Examples

```
python trimmed2.py --fastq_dir tests/trimmed_fastq/
```

## Evaluation signals

- Trimmed FASTQ read names carry appended SSS barcode and IVT UMI sequences extracted during barcode matching
- First-round 8-nt Tn5 and second-round 7-nt ligation barcodes each matched with at most 1 mismatch (Levenshtein distance)
- No residual Tn5 MEDS adapter sequence remains in the trimmed read sequences before alignment
- trimmed2.py successfully produces the trimmed. index file enumerating per-sample trimmed FASTQs
- Read pair orientation resolved via RT primer search with rt_mismatches 3 and third-round barcode detection, with sss_mismatches 0 for the SSS barcode

## Limitations

- Mismatch tolerances are fixed by config.yaml (tn5_mismatches 1, ligation_mismatches 1); reads whose barcodes exceed these tolerances cannot be assigned single-cell combinatorial barcode identities
- Low RNA read recovery per cell is a known limitation of the co-assay, so the RNA-versus-DNA tag (rt_UMI versus GGGGGG signature) may be sparse for some cells
- Additional rounds of cell barcoding under the sci framework reduce the number of recovered single cells due to loss of per-cell coverage

## Evidence

- [methods] Trim Tn5 MEDS adapter sequence from the reads using cutadapt.: [evidence span withheld — anchor status: not_found]
- [methods] trimmed2.py indexes annotated trimmed FASTQs: [evidence span withheld — anchor status: not_found]
- [other] read orientation and barcode extraction step: [evidence span withheld — anchor status: not_found]
