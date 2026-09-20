---
name: single-cell-bam-quality-filtering
description: Use when processing split single-cell BAMs from a sci-L3-Strand-seq run
  and you must decide which sorted cells pass strand-state QC before downstream SCE
  counting, phasing, or coverage aggregation.
when_to_use_negative:
- Data is sci-L3-Hi-C or RNA/ATAC rather than Strand-seq — WC/uncategorized and background
  strand-state criteria are Strand-seq specific.
- Input is raw FASTQ or unsplit sample-level BAMs; cells must already be split into
  single-cell BAMs before per-cell QC applies.
- The goal is library-level (not cell-level) quality control such as barcode mismatch
  filtering (sss_mismatches, rt_mismatches, tn5_mismatches, ligation_mismatches).
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: breakpointR
  role: Segments strand states and calls sister chromatid exchanges per chromosome
    from single-cell BAMs (windowSize 5e6, background 0.05, minReads 50), providing
    the metrics the filter evaluates
- name: sss_bam2.py
  role: Builds the SSS BAM inventory that feeds the split single-cell BAMs used for
    per-cell QC
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: Snakemake
  role: Orchestrates the split_bam step that produces the per-cell BAMs and downstream
    QC workflow
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: bwa
  role: Aligner producing the SSS BAMs that are split into single-cell BAMs prior
    to filtering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:28d075f151b1545c8c8edf695c57473329d3f2f7e2394780ee877efad2298daf
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
  - breakpointR
  - bwa
  - sss_bam2.py
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# single-cell-bam-quality-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the sci-L3-Strand-seq cell-level QC filter after breakpointR strand-state calling: retain single-cell BAMs whose WC/uncategorized fraction (<=15% haploid, <=75% diploid) and background (0–0.08) meet the reported thresholds, then compute pass rate, per-cell SCE counts, and relative chromosome coverage.

## When to use

Use when processing split single-cell BAMs from a sci-L3-Strand-seq run and you must decide which sorted cells pass strand-state QC before downstream SCE counting, phasing, or coverage aggregation.

## When NOT to use

- Data is sci-L3-Hi-C or RNA/ATAC rather than Strand-seq — WC/uncategorized and background strand-state criteria are Strand-seq specific.
- Input is raw FASTQ or unsplit sample-level BAMs; cells must already be split into single-cell BAMs before per-cell QC applies.
- The goal is library-level (not cell-level) quality control such as barcode mismatch filtering (sss_mismatches, rt_mismatches, tn5_mismatches, ligation_mismatches).

## Inputs

- Split single-cell BAM files from the split_bam step of the sciL3Pipe Snakemake pipeline
- BAM inventory produced by sss_bam2.py (sss_bam2.py --bam_dir ...)
- Per-cell breakpointR strand-state segments, background estimates, and SCE calls
- Cell ploidy labels (haploid vs diploid) for each cell line

## Outputs

- breakpointR_sce_summary.tsv with per-cell strand-state composition, SCE counts, pass/fail status, and chromosome coverage
- QC pass-rate statistic (fraction of sorted cells passing filter)
- Per-cell-line mean SCE counts (expected 4 +/- 2.7 HAP1, 7 +/- 2.8 Patski)

## How to apply

First run breakpointR on each split single-cell BAM (produced by sss_bam2.py inventory plus the split_bam Snakemake step) with windowSize = 5e6, background = 0.05, and minReads = 50 to segment strand states and call sister chromatid exchanges per chromosome. Then apply the paper's quality criteria per cell: WC/uncategorized fraction <= 15% for haploid cells (e.g. HAP1) or <= 75% for diploid cells (e.g. Patski), and background between 0 and 0.08. Cells meeting both criteria are marked pass; compute the pass rate among all sorted cells and compare to the reported >58% (with ~0.22% average genome coverage per passing cell). Aggregate per-cell SCE counts by cell line, expecting 4 +/- 2.7 SCEs in HAP1 and 7 +/- 2.8 in Patski. Write per-cell strand-state composition, SCE counts, pass/fail status, and chromosome coverage to breakpointR_sce_summary.tsv. A pre-sequencing read cutoff (e.g. cutoff: 2000 reads per single cell, default 10000) also gates which cells are processed at all.

## Related tools

- **breakpointR** (Segments strand states and calls sister chromatid exchanges per chromosome from single-cell BAMs (windowSize 5e6, background 0.05, minReads 50), providing the metrics the filter evaluates)
- **sss_bam2.py** (Builds the SSS BAM inventory that feeds the split single-cell BAMs used for per-cell QC) — https://github.com/recombinationlab/sciL3Pipe
- **Snakemake** (Orchestrates the split_bam step that produces the per-cell BAMs and downstream QC workflow) — https://github.com/recombinationlab/sciL3Pipe
- **bwa** (Aligner producing the SSS BAMs that are split into single-cell BAMs prior to filtering)

## Examples

```
python sss_bam2.py --bam_dir tests/bwa_alignment/ && snakemake --configfile configs/config.yaml
```

## Evaluation signals

- Reproduced QC pass rate exceeds 58% of sorted sci-L3-Strand-seq cells with strand information
- Mean SCE counts fall within reported ranges: 4 +/- 2.7 (HAP1) and 7 +/- 2.8 (Patski)
- Every passing cell satisfies WC/uncategorized fraction <= 15% (haploid) or <= 75% (diploid) and background in [0, 0.08]
- Average genome coverage per passing cell is ~0.22%
- breakpointR_sce_summary.tsv contains per-cell strand-state composition, SCE counts, pass/fail status, and chromosome coverage columns

## Limitations

- Adding additional rounds of cell barcoding under the sci framework reduces per-cell coverage and the number of recovered single cells, shrinking the passing set
- No software version numbers are stated for the pipeline/Docker images, complicating exact reproduction of QC metrics
- The cell-sorted library only enriches for BrdU-labeled cells of the right cell cycle, so pass rates depend on the sorting step upstream of QC

## Evidence
