---
name: snv-calling-from-bulk-wgs
description: Use when a cell-mixing single-cell experiment needs a ground-truth cell
  identity ground truth derived from cell line-specific SNVs, and a bulk WGS sample
  (FASTQ/BAM) is available for one of the mixed lines.
when_to_use_negative:
- The single-cell reads themselves are the intended source for de novo SNV discovery
  — low per-cell coverage (sci-L3 linear-amplification data) is unsuitable for calling
  SNVs per cell; bulk WGS must supply the variant panel.
- Cell identity can be established directly from experimental labeling or sorting
  rather than genotype — a SNV-rate based assignment adds unnecessary complexity.
- The bulk WGS and single-cell data are aligned to different genome assemblies without
  performing coordinate liftover (e.g., HEK293T VCF in hg18 vs hg19 single-cell alignments)
  — intersecting without lifting will yield spurious 'private' variant lists.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bwa
  role: 'Aligner used to map the bulk WGS FASTQs to hg19, matching the sci-L3 pipeline
    config (aligner: ''bwa'')'
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: bcftools
  role: mpileup/call used to call SNVs from the aligned bulk BAM, filtered to high-confidence
    variants
- name: sss_bam2.py
  role: Splits sci-L3 SSS BAMs into per-single-cell BAMs on which the private-SNV
    allele counts are computed
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: Snakemake
  role: Orchestrates the sci-L3 processing pipeline whose split single-cell BAMs feed
    the SNV-rate assignment
  repo: https://github.com/recombinationlab/sciL3Pipe
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:14eaa6f253862b3182bbf0e05dc5980daa1fa48e61c2f0929bccebe69c0c8037
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# snv-calling-from-bulk-wgs

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Call high-confidence SNVs from bulk whole-genome sequencing (WGS) of a reference cell line (e.g., BJ from SRA SRP102259, aligned to hg19 with bwa) to build a panel of line-private SNVs that can then be used to assign single-cell identities in sci-L3 experiments.

## When to use

Use when a cell-mixing single-cell experiment needs a ground-truth cell identity ground truth derived from cell line-specific SNVs, and a bulk WGS sample (FASTQ/BAM) is available for one of the mixed lines.

## When NOT to use

- The single-cell reads themselves are the intended source for de novo SNV discovery — low per-cell coverage (sci-L3 linear-amplification data) is unsuitable for calling SNVs per cell; bulk WGS must supply the variant panel.
- Cell identity can be established directly from experimental labeling or sorting rather than genotype — a SNV-rate based assignment adds unnecessary complexity.
- The bulk WGS and single-cell data are aligned to different genome assemblies without performing coordinate liftover (e.g., HEK293T VCF in hg18 vs hg19 single-cell alignments) — intersecting without lifting will yield spurious 'private' variant lists.

## Inputs

- Bulk WGS FASTQ files for a reference cell line (e.g., BJ bulk WGS, SRA SRP102259)
- Human reference genome assembly (hg19 for the BJ mixing experiment; hg38/GRCh38 in the pipeline config)
- Published VCF of a second cell line (e.g., HEK293T, in hg18 coordinates)
- Single-cell BAMs from the sci-L3 pipeline split per cell (sss_bam2.py output)

## Outputs

- High-confidence bulk SNV calls (bcftools VCF) for the reference line
- Line-private SNV lists (e.g., BJ-private vs HEK293T-private variants) after coordinate liftover and intersection
- Per-cell cell-line assignments (e.g., single_cell_snv_assignments.tsv) with SNV-collision cells excluded

## How to apply

Download the bulk WGS FASTQs (e.g., BJ bulk from SRA SRP102259) and align them to hg19 with bwa, matching the sci-L3 pipeline's aligner setting (aligner: 'bwa'). Call SNVs from the aligned bulk BAM using bcftools mpileup/call and filter to high-confidence variants. To obtain a second line-private panel (e.g., HEK293T), lift the published VCF from hg18 to hg19 with a coordinate lifter and intersect it with the BJ calls to derive BJ-private versus HEK293T-private SNV lists. These private-SNV lists are then applied against single-cell BAMs split out by the sci-L3 pipeline (split_bam: True via sss_bam2.py, human chromosomes only, additional_filter 'hs37d5'): per cell, count reads supporting each line's private alleles and assign the cell to a line when >70% of informative SNV reads support one line, otherwise flag it as a SNV collision and exclude it.

## Related tools

- **bwa** (Aligner used to map the bulk WGS FASTQs to hg19, matching the sci-L3 pipeline config (aligner: 'bwa')) — https://github.com/recombinationlab/sciL3Pipe
- **bcftools** (mpileup/call used to call SNVs from the aligned bulk BAM, filtered to high-confidence variants)
- **sss_bam2.py** (Splits sci-L3 SSS BAMs into per-single-cell BAMs on which the private-SNV allele counts are computed) — https://github.com/recombinationlab/sciL3Pipe
- **Snakemake** (Orchestrates the sci-L3 processing pipeline whose split single-cell BAMs feed the SNV-rate assignment) — https://github.com/recombinationlab/sciL3Pipe

## Examples

```
python sss_bam2.py --bam_dir tests/bwa_alignment/
```

## Evaluation signals

- Per-cell SNV-rate assignment recapitulates expected mixing fractions: 383/390 cells unambiguously assigned to a cell line (only a small residual of SNV collisions)
- Bulk SNV VCF contains only high-confidence variants after bcftools filtering (no low-depth/low-quality calls retained)
- BJ-private vs HEK293T-private SNV lists are non-overlapping and map correctly to hg19 after liftover of the hg18 HEK293T VCF
- Single-cell BAMs were split with the expected pipeline flags (split_bam: True; additional_filter 'hs37d5'; human chromosomes only)
- Cells assigned via the >70% private-SNV-rate threshold show consistent assignments across RNA and ATAC modalities of the sci-L3-RNA/ATAC co-assay

## Limitations

- Cells whose private-SNV read rate falls at or below 70% cannot be assigned and are excluded as SNV collisions, discarding some cells from downstream analysis.
- The second cell line's variants may be published on a different assembly (HEK293T hg18 VCF), requiring coordinate liftover that can introduce mapping errors at lifted sites.
- Per-cell coverage in sci-L3 libraries is limited (default reads-per-cell cutoff 2000 in the pipeline config), so informative SNV reads per cell are sparse and the assignment depends on sufficient coverage of private loci.
- No software version numbers are stated for the pipeline's docker/apptainer container, complicating exact reproduction of the calling step.

## Evidence

- [results] Per-cell cell-line assignment by private-SNV rate with >70% threshold: [evidence span withheld — anchor status: not_found]
- [methods] Pipeline aligner configuration: [evidence span withheld — anchor status: not_found]
