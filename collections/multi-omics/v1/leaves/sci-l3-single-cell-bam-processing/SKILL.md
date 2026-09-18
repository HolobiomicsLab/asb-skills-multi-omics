---
name: sci-l3-single-cell-bam-processing
description: Use when single-cell BAMs from the sci-L3-RNA/ATAC human cell mixing
  experiment (BJ vs HEK293T) need cell identity labels, and private-SNV VCFs (BJ from
  bulk WGS, HEK293T lifted to hg19) are available as the ground-truth reference.
when_to_use_negative:
- Single cells come from a species or cell lines without pre-existing line-specific
  SNV references, so no private-SNV denominator exists to compute a >70% assignment
  rate.
- The goal is demultiplexing by genetic barcodes or hashing rather than endogenous
  private-SNV rates; this SNV-collision workflow would mislabel multiplets instead
  of excluding them.
- 'Reads per cell fall below the pipeline cutoff (cutoff: 2000 in the example config),
  making per-cell SNV-rate estimates too sparse to be informative.'
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: bwa
  role: aligns BJ bulk WGS reads to hg19 to generate the SNV-calling input BAM
- name: bcftools
  role: mpileup/call variant calling from the bulk BAM to obtain high-confidence BJ
    SNVs
- name: sss_bam2.py
  role: splits SSS BAMs into single-cell BAMs for per-cell SNV counting
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: Python
  role: computes per-cell cell-line-specific SNV rates and applies the >70% assignment
    threshold
- name: Snakemake
  role: drives the sciL3Pipe workflow including the split_bam and filtering steps
  repo: https://github.com/recombinationlab/sciL3Pipe
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:4d9b4dd5995b3203d236e3597484e7420a07943683fd3f323cf6abc674d32f6e
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
  - bcftools
  - bwa
  - sss_bam2.py
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# sci-l3-single-cell-bam-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Derives per-cell cell-line identity from single-cell sci-L3 BAMs by counting reads supporting cell line-specific SNVs and applying a >70% SNV-rate assignment threshold, so doublet/collision cells are excluded from downstream analysis.

## When to use

Use when single-cell BAMs from the sci-L3-RNA/ATAC human cell mixing experiment (BJ vs HEK293T) need cell identity labels, and private-SNV VCFs (BJ from bulk WGS, HEK293T lifted to hg19) are available as the ground-truth reference.

## When NOT to use

- Single cells come from a species or cell lines without pre-existing line-specific SNV references, so no private-SNV denominator exists to compute a >70% assignment rate.
- The goal is demultiplexing by genetic barcodes or hashing rather than endogenous private-SNV rates; this SNV-collision workflow would mislabel multiplets instead of excluding them.
- Reads per cell fall below the pipeline cutoff (cutoff: 2000 in the example config), making per-cell SNV-rate estimates too sparse to be informative.

## Inputs

- BJ bulk WGS FASTQs from SRA SRP102259
- Published HEK293T VCF in hg18 coordinates
- sci-L3 single-cell SSS BAMs produced by sss_bam2.py (split_bam: True)
- human reference genome hg19 with hs37d5 additional filter

## Outputs

- single_cell_snv_assignments.tsv with per-cell cell-line identity or SNV collision labels
- BJ-private and HEK293T-private SNV lists from intersecting BJ bulk calls with lifted HEK293T VCF

## How to apply

First build the private-SNV reference: download BJ bulk WGS FASTQs from SRA SRP102259, align to hg19 with bwa (the pipeline's aligner: 'bwa'), call SNVs with the reported bcftools mpileup/call command, lift the published HEK293T VCF from hg18 to hg19, and intersect to derive BJ-private and HEK293T-private SNV lists. Then process single cells with the sciL3Pipe Snakemake workflow: run sss_bam2.py with split_bam: True, processing only human chromosomes (process_only: 'human') with additional_filter 'hs37d5', keeping cells passing the reads-per-cell cutoff (2000 in the example config). For each single-cell BAM, count reads supporting BJ-specific versus HEK293T-private SNV alleles and compute the per-cell line-specific SNV rate; assign a cell to a line only when >70% of informative SNV reads support one line, otherwise label it an SNV collision and exclude it from downstream analysis.

## Related tools

- **bwa** (aligns BJ bulk WGS reads to hg19 to generate the SNV-calling input BAM)
- **bcftools** (mpileup/call variant calling from the bulk BAM to obtain high-confidence BJ SNVs)
- **sss_bam2.py** (splits SSS BAMs into single-cell BAMs for per-cell SNV counting) — https://github.com/recombinationlab/sciL3Pipe
- **Python** (computes per-cell cell-line-specific SNV rates and applies the >70% assignment threshold)
- **Snakemake** (drives the sciL3Pipe workflow including the split_bam and filtering steps) — https://github.com/recombinationlab/sciL3Pipe

## Examples

```
python sss_bam2.py --bam_dir tests/bwa_alignment/
```

## Evaluation signals

- 383/390 cells are unambiguously assigned to a cell line in single_cell_snv_assignments.tsv.
- Every assigned cell has >70% of informative SNV reads supporting a single line; remaining cells are labeled SNV collisions and excluded from downstream analysis.
- Single-cell BAM splitting used split_bam: True with additional_filter 'hs37d5' and process_only: 'human', matching the published config.
- Per-cell read counts respect the reads-per-single-cell cutoff (2000 in the example config).
- BJ-private/HEK293T-private SNV lists derive from bulk bcftools calls intersected with the hg18→hg19 lifted HEK293T VCF.

## Limitations

- Cells whose SNV rate falls at or below 70% are classified as SNV collisions and excluded, reducing the number of usable cells downstream.
- A limitation of the current sci-L3-RNA/ATAC approach is the low recovery of RNA reads per cell, and adding additional rounds of cell barcoding reduces the number of recovered single cells due to loss of per-cell coverage.
- The pipeline repository states no software version numbers and pulls docker containers automatically, so exact tool versions for the SNV workflow are not pinned.

## Evidence

- [methods] Per-cell SNV assignment threshold: [evidence span withheld — anchor status: not_found]
- [methods] sss_bam2.py invocation: [evidence span withheld — anchor status: not_found]
