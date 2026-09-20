---
name: vcf-intersection-private-variant-derivation
description: Use when demultiplexing single cells from a mixed-cell-line sci-L3 experiment
  (e.g. BJ + HEK293T RNA/ATAC) into per-cell identities using cell line-specific SNV
  rates derived from intersected VCFs.
when_to_use_negative:
- The experiment is not a mixture of genotypically distinct cell lines (no line-private
  SNV set can be derived, so identity cannot be assigned from SNV rates).
- Cells are already demultiplexed or an external cell-identity label exists, making
  SNV-based assignment redundant.
- 'Only non-human contigs are of interest — the assignment workflow processes only
  human chromosomes (process_only: "human").'
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3168
tools:
- name: bwa
  role: Aligns BJ bulk WGS FASTQs to hg19, matching the pipeline's aligner setting
  repo: http://bio-bwa.sourceforge.net/
- name: bcftools
  role: Calls SNVs from the aligned bulk BAM via mpileup/call to build the BJ variant
    set
- name: Python
  role: Counts per-cell reads supporting line-specific SNV alleles, computes SNV rates,
    and applies the >70% assignment rule
- name: Snakemake
  role: Runs the sciL3Pipe pipeline (configfile with aligner, filters, split_bam)
    that yields the single-cell BAMs used for SNV counting
  repo: https://github.com/recombinationlab/sciL3Pipe
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:db7094f17192e962e622e390a2b1fdf8b5fa0c1d0aef19818f9a1e704de2aa26
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# vcf-intersection-private-variant-derivation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Derives cell-line-private SNV lists by intersecting bulk WGS-called (BJ) and published (HEK293T, hg18→hg19 lifted) VCFs, then assigns each single cell in a mixed sci-L3-RNA/ATAC experiment to a cell line from the rate of line-specific SNV-supporting reads, using a >70% threshold and labeling ambiguous cells as SNV collisions.

## When to use

Use when demultiplexing single cells from a mixed-cell-line sci-L3 experiment (e.g. BJ + HEK293T RNA/ATAC) into per-cell identities using cell line-specific SNV rates derived from intersected VCFs.

## When NOT to use

- The experiment is not a mixture of genotypically distinct cell lines (no line-private SNV set can be derived, so identity cannot be assigned from SNV rates).
- Cells are already demultiplexed or an external cell-identity label exists, making SNV-based assignment redundant.
- Only non-human contigs are of interest — the assignment workflow processes only human chromosomes (process_only: "human").

## Inputs

- BJ bulk WGS FASTQs from SRA SRP102259
- Published HEK293T VCF (hg18 coordinates)
- sci-L3 split single-cell BAMs produced by sss_bam2.py
- Human reference genome hg19 with hs37d5 additional filter

## Outputs

- BJ-private and HEK293T-private SNV lists (hg19)
- single_cell_snv_assignments.tsv with per-cell line identity and SNV rate

## How to apply

Download BJ bulk WGS FASTQs (SRA SRP102259) and align them to hg19 with bwa (aligner: 'bwa' in the sciL3Pipe config), then call SNVs from the aligned bulk BAM with the reported bcftools mpileup/call command and keep high-confidence variants. Lift the published HEK293T VCF from hg18 to hg19 coordinates and intersect it with the BJ SNV calls to derive BJ-private and HEK293T-private SNV lists. Split the sci-L3 pipeline output into single-cell BAMs (split_bam: True via sss_bam2.py, processing only human chromosomes with additional_filter 'hs37d5'), then for each cell count reads supporting BJ-specific versus HEK293T-specific alleles and compute the per-cell line-specific SNV rate. Assign a cell to a line when >70% of informative SNV reads support that line; otherwise classify it as a SNV collision and exclude it from downstream analysis, writing results to single_cell_snv_assignments.tsv.

## Related tools

- **bwa** (Aligns BJ bulk WGS FASTQs to hg19, matching the pipeline's aligner setting) — http://bio-bwa.sourceforge.net/
- **bcftools** (Calls SNVs from the aligned bulk BAM via mpileup/call to build the BJ variant set)
- **Python** (Counts per-cell reads supporting line-specific SNV alleles, computes SNV rates, and applies the >70% assignment rule)
- **Snakemake** (Runs the sciL3Pipe pipeline (configfile with aligner, filters, split_bam) that yields the single-cell BAMs used for SNV counting) — https://github.com/recombinationlab/sciL3Pipe

## Examples

```
snakemake --configfile configs/config.yaml
```

## Evaluation signals

- The derived BJ-private and HEK293T-private SNV lists are non-empty and mutually exclusive after hg18→hg19 lift and intersection.
- The per-cell SNV-rate distribution is bimodal, with most cells near 0% or 100% for one line's private alleles.
- The number of unambiguously assigned cells matches the expected 383/390, with the remainder flagged as SNV collisions (<70% rate).
- single_cell_snv_assignments.tsv exists, has one row per cell, and every assigned cell exceeds the 70% threshold.
- Single-cell BAM splitting used the expected settings (split_bam: True, additional_filter: 'hs37d5', process_only: 'human').

## Limitations

- Cells failing the >70% line-specific SNV rate cannot be assigned and are excluded as SNV collisions, so a small fraction of cells (7/390 in the reference run) is lost.
- Assignment relies on an external, previously published HEK293T VCF that must be lifted from hg18 to hg19; coordinate-conversion errors propagate into the private-SNV lists.
- Only human chromosomes are processed (process_only: 'human', additional_filter 'hs37d5'), and the pipeline discards cells below the reads-per-cell cutoff (default 10000, config example cutoff: 2000), which can reduce the set of assignable cells.

## Evidence

- [results] Per-cell cell-line assignment via >70% private-SNV rate: [evidence span withheld — anchor status: not_found]
- [readme] sss_bam2.py produces the split single-cell BAMs used for SNV counting: [evidence span withheld — anchor status: not_found]
