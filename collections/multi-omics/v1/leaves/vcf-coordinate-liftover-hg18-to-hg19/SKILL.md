---
name: vcf-coordinate-liftover-hg18-to-hg19
description: Use when a published VCF is on an older assembly (hg18) than the working
  reference (hg19) and must be coordinate-converted and intersected with new SNV calls
  to build sample-private variant lists.
when_to_use_negative:
- Both variant callsets are already on the same reference assembly (hg19), so no coordinate
  conversion is needed — intersect directly.
- The goal is de novo SNV calling in a single sample rather than reconciling a legacy
  hg18 callset against newly called variants.
- The cells of interest are not from a mixing experiment with line-private SNV markers
  available, so the private-SNV rate logic does not apply.
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: bwa
  role: 'Aligns BJ bulk WGS FASTQs to the hg19 reference before SNV calling (aligner:
    ''bwa'')'
  repo: http://bio-bwa.sourceforge.net/
- name: bcftools
  role: Calls SNVs from the aligned bulk BAM using the reported mpileup/call command
    to generate the hg19 BJ SNV set
- name: Python
  role: Performs the VCF liftover and intersection logic and computes per-cell SNV
    rates for the >70% assignment rule
- name: Snakemake (sciL3Pipe)
  role: Reproducible pipeline wrapper providing the aligned single-cell BAMs consumed
    by the SNV-rate assignment step
  repo: https://github.com/recombinationlab/sciL3Pipe
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:2d6cfa53d0ab41fa93cbf9ea1845581f25c435c344eb8b6e70d4717346d19736
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
  - Snakemake (sciL3Pipe)
  - bcftools
  - bwa
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# vcf-coordinate-liftover-hg18-to-hg19

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Lift a published variant callset (HEK293T VCF in hg18 coordinates) onto hg19 and intersect it with newly called SNVs from BJ bulk WGS so that BJ-private and HEK293T-private SNV lists can be derived for per-cell line-identity assignment in the sci-L3-RNA/ATAC mixing experiment.

## When to use

Use when a published VCF is on an older assembly (hg18) than the working reference (hg19) and must be coordinate-converted and intersected with new SNV calls to build sample-private variant lists.

## When NOT to use

- Both variant callsets are already on the same reference assembly (hg19), so no coordinate conversion is needed — intersect directly.
- The goal is de novo SNV calling in a single sample rather than reconciling a legacy hg18 callset against newly called variants.
- The cells of interest are not from a mixing experiment with line-private SNV markers available, so the private-SNV rate logic does not apply.

## Inputs

- Published HEK293T VCF in hg18 coordinates
- BJ bulk WGS FASTQs from SRA SRP102259
- hg19 human reference genome for bwa alignment
- Single-cell BAMs produced by sss_bam2.py from the sci-L3 pipeline (split_bam: True)

## Outputs

- BJ-private SNV list (hg19 coordinates)
- HEK293T-private SNV list (hg19 coordinates)
- Per-cell cell-line-specific SNV rates
- single_cell_snv_assignments.tsv with per-cell line identities and SNV-collision calls

## How to apply

First download the BJ bulk WGS FASTQs from SRA SRP102259 and align them to hg19 with bwa, matching the sci-L3 pipeline's aligner setting (aligner: 'bwa'), then call SNVs from the bulk BAM using the exact reported bcftools mpileup/call command and filter to high-confidence variants. Next, lift the published HEK293T VCF from hg18 to hg19 coordinates and intersect it with the BJ SNV calls to derive mutually exclusive BJ-private and HEK293T-private SNV lists; the rationale is that only line-private variants are informative markers for assigning single cells to a mixing-experiment cell line. These private SNV lists are then used downstream: for each single cell (from split single-cell BAMs produced with split_bam: True and sss_bam2.py, processing only human chromosomes with additional_filter 'hs37d5'), count reads supporting BJ-specific versus HEK293T-specific alleles and compute the per-cell cell-line-specific SNV rate. A cell is assigned to a cell line when >70% of informative SNV reads support one line; otherwise it is classified as a SNV collision and excluded from downstream analysis. The decision point to verify is that the liftover produced correctly positioned hg19 variants — a mis-lift would corrupt the private-SNV intersection and inflate the SNV-collision rate.

## Related tools

- **bwa** (Aligns BJ bulk WGS FASTQs to the hg19 reference before SNV calling (aligner: 'bwa')) — http://bio-bwa.sourceforge.net/
- **bcftools** (Calls SNVs from the aligned bulk BAM using the reported mpileup/call command to generate the hg19 BJ SNV set)
- **Python** (Performs the VCF liftover and intersection logic and computes per-cell SNV rates for the >70% assignment rule)
- **Snakemake (sciL3Pipe)** (Reproducible pipeline wrapper providing the aligned single-cell BAMs consumed by the SNV-rate assignment step) — https://github.com/recombinationlab/sciL3Pipe

## Evaluation signals

- The lifted HEK293T VCF positions map to valid hg19 coordinates and the intersection yields mutually exclusive (private) BJ and HEK293T variant lists with no overlapping entries.
- Per-cell SNV-rate computation assigns a cell only when >70% of informative SNV reads support a single cell line; anything below is written out as a SNV collision.
- Of the 390 cells in the sci-L3-RNA/ATAC mixing experiment, 383 cells should be unambiguously assigned in single_cell_snv_assignments.tsv.
- The bcftools mpileup/call command matches the exact reported invocation, and the bulk alignment used bwa against hg19 as in the pipeline config.
- Single-cell BAM inputs were produced with split_bam: True and additional_filter 'hs37d5', processing only human chromosomes.

## Limitations

- Cells whose cell-line-specific SNV rate does not exceed 70% are classified as SNV collisions and excluded from downstream analysis, so borderline cells are lost.
- hg18-to-hg19 coordinate conversion can silently misplace variants if the liftover is not validated, corrupting the private-SNV intersection.
- The sci-L3 pipeline context states no software version numbers are documented for the tools, complicating exact reproduction.

## Evidence

- [methods] Liftover to hg19: [evidence span withheld — anchor status: not_found]
- [results] SNV rate assignment rule: [evidence span withheld — anchor status: not_found]
- [methods] Bulk SNV calling: [evidence span withheld — anchor status: not_found]
- [readme] Single-cell BAM production: [evidence span withheld — anchor status: not_found]
