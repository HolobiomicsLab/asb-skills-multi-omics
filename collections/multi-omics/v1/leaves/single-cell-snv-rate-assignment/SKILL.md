---
name: single-cell-snv-rate-assignment
description: Use when single cells come from a mixing experiment between lines with
  known genetic differences (e.g. BJ vs HEK293T) and each cell's line identity must
  be established from SNV support rates rather than barcodes.
when_to_use_negative:
- Cells are not from a line-mixing experiment with pre-existing bulk WGS or a published
  VCF defining line-private SNVs (no private-variant reference can be built).
- 'Data are non-human or the sci-L3 pipeline was not run with split_bam: True and
  the hs37d5 additional filter, since the assignment operates on human-chromosome
  single-cell BAMs only.'
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: bwa
  role: Aligns BJ bulk WGS FASTQs to hg19 to build the line-private SNV reference
  repo: github.com/recombinationlab/sciL3Pipe
- name: bcftools
  role: mpileup/call SNV variant calling from the aligned bulk BAM
- name: Python
  role: Counts per-cell BJ- vs HEK293T-specific SNV reads, computes the >70% rate,
    and writes single_cell_snv_assignments.tsv
  repo: github.com/recombinationlab/sciL3Pipe
- name: Snakemake
  role: Orchestrates the sci-L3 pipeline whose split_bam/sss_bam2.py step produces
    the per-cell BAMs used for SNV counting
  repo: github.com/recombinationlab/sciL3Pipe
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:1eee37e10c22b9dc443a0600af76ff2ca628b6638b67ca6efb2ac3dc497ac11d
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
  - bcftools
  - bwa
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# single-cell-snv-rate-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Demultiplexes single cells from a cell-mixing experiment into their originating cell lines by counting reads supporting line-private SNVs, requiring >70% of informative SNV reads to support one line before assigning identity.

## When to use

Use when single cells come from a mixing experiment between lines with known genetic differences (e.g. BJ vs HEK293T) and each cell's line identity must be established from SNV support rates rather than barcodes.

## When NOT to use

- Cells are not from a line-mixing experiment with pre-existing bulk WGS or a published VCF defining line-private SNVs (no private-variant reference can be built).
- Data are non-human or the sci-L3 pipeline was not run with split_bam: True and the hs37d5 additional filter, since the assignment operates on human-chromosome single-cell BAMs only.

## Inputs

- BJ bulk WGS FASTQs from SRA SRP102259
- hg19 human reference genome for bwa alignment
- Published HEK293T VCF in hg18 coordinates (lifted to hg19)
- Split single-cell BAMs produced by sss_bam2.py from sci-L3 SSS BAMs

## Outputs

- High-confidence BJ-private and HEK293T-private SNV lists
- Per-cell cell-line-specific SNV rate
- single_cell_snv_assignments.tsv mapping each cell to BJ or HEK293T or flagging SNV collision

## How to apply

First build the reference variant sets: download BJ bulk WGS FASTQs from SRA SRP102259, align to hg19 with bwa (matching the pipeline's aligner: 'bwa' setting), call SNVs with the reported bcftools mpileup/call command, lift the published HEK293T VCF from hg18 to hg19, and intersect to derive BJ-private and HEK293T-private SNV lists. Next obtain per-cell BAMs from the sci-L3 pipeline by running sss_bam2.py with split_bam: True, processing only human chromosomes with additional_filter 'hs37d5'. For each single cell, count reads supporting BJ-specific versus HEK293T-specific SNV alleles and compute the per-cell cell-line-specific SNV rate. Assign a cell to a line only when >70% of informative SNV reads support that line; cells below the threshold are classified as SNV collisions and excluded from downstream analysis.

## Related tools

- **bwa** (Aligns BJ bulk WGS FASTQs to hg19 to build the line-private SNV reference) — github.com/recombinationlab/sciL3Pipe
- **bcftools** (mpileup/call SNV variant calling from the aligned bulk BAM)
- **Python** (Counts per-cell BJ- vs HEK293T-specific SNV reads, computes the >70% rate, and writes single_cell_snv_assignments.tsv) — github.com/recombinationlab/sciL3Pipe
- **Snakemake** (Orchestrates the sci-L3 pipeline whose split_bam/sss_bam2.py step produces the per-cell BAMs used for SNV counting) — github.com/recombinationlab/sciL3Pipe

## Examples

```
python sss_bam2.py --bam_dir tests/bwa_alignment/
```

## Evaluation signals

- 383 of 390 cells receive an unambiguous cell-line assignment in single_cell_snv_assignments.tsv
- Every assigned cell has >70% of informative SNV reads supporting one line; remaining cells are labeled SNV collisions and excluded
- BJ-private and HEK293T-private SNV lists are non-overlapping after intersecting bulk BJ calls with the hg18→hg19-lifted HEK293T VCF
- Per-cell BAMs derive only from the human-chromosome SSS split (additional_filter 'hs37d5', split_bam True)
- Bulk SNV calls come from bcftools mpileup/call on bwa-aligned BJ WGS against hg19

## Limitations

- Cells failing the >70% SNV-rate threshold are treated as SNV collisions and discarded, reducing the set of usable single cells.
- The ground truth depends on external variant resources (BJ bulk WGS from SRA SRP102259 and a published HEK293T VCF that must be lifted from hg18 to hg19); errors or coordinate-conversion mistakes propagate into assignments.
- Pipeline-level reproducibility gaps: no software version numbers are stated and no changelog exists for the associated repository.

## Evidence

- [full_text] Per-cell SNV rate threshold and collision handling: [evidence span withheld — anchor status: not_found]
- [methods] Per-cell BAM splitting step feeding the assignment: [evidence span withheld — anchor status: not_found]
