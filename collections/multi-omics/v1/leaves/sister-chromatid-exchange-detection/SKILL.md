---
name: sister-chromatid-exchange-detection
description: Use when processing split single-cell Strand-seq BAMs (e.g., sci-L3-Strand-seq
  output from the split_bam step) to segment strand states with breakpointR, call
  SCEs per cell, and apply reported QC thresholds.
when_to_use_negative:
- Input is bulk Strand-seq or non-single-cell BAM data — the workflow operates on
  split single-cell BAMs from combinatorial indexing sci-L3 libraries
- The goal is Hi-C compartment or interaction analysis from sci-L3-Hi-C data — this
  skill covers Strand-seq strand-state/SCE analysis only
- Cells were not sorted/enriched for BrdU-labeled cells of the right cell cycle, since
  template-strand informativeness depends on that enrichment step
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3678
- http://edamontology.org/topic_3308
tools:
- name: breakpointR
  role: Segments strand states and calls sister chromatid exchanges per chromosome
    from single-cell Strand-seq BAMs (windowSize 5e6, background 0.05, minReads 50)
- name: Snakemake
  role: Orchestrates the sciL3Pipe workflow including the split_bam step that produces
    per-cell BAMs; invoked via snakemake --configfile configs/config.yaml
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: bwa
  role: Aligns sci-L3-Strand-seq reads to the hg38 (GRCh38) reference prior to single-cell
    BAM splitting
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: sss_bam2.py
  role: Builds the SSS BAM inventory consumed downstream for splitting into single-cell
    BAMs
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: Python
  role: Drives helper scripts (fastq2.py, trimmed2.py, sss_bam2.py) and downstream
    aggregation of per-cell SCE and QC metrics
  repo: https://github.com/recombinationlab/sciL3Pipe
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:9736a6743fdf282549611b2d9ad4f07b9f0124aee0648d12401cf85741c9e8b8
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3678
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# sister-chromatid-exchange-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detection and quantification of sister chromatid exchanges (SCEs) from sci-L3-Strand-seq single-cell BAMs by running breakpointR strand-state segmentation and QC filtering, as demonstrated by Chovanec & Yin (2025) who detect SCEs in thousands of single cells.

## When to use

Use when processing split single-cell Strand-seq BAMs (e.g., sci-L3-Strand-seq output from the split_bam step) to segment strand states with breakpointR, call SCEs per cell, and apply reported QC thresholds.

## When NOT to use

- Input is bulk Strand-seq or non-single-cell BAM data — the workflow operates on split single-cell BAMs from combinatorial indexing sci-L3 libraries
- The goal is Hi-C compartment or interaction analysis from sci-L3-Hi-C data — this skill covers Strand-seq strand-state/SCE analysis only
- Cells were not sorted/enriched for BrdU-labeled cells of the right cell cycle, since template-strand informativeness depends on that enrichment step

## Inputs

- Split single-cell BAM files from the split_bam step of the Snakemake pipeline (bwa-aligned to hg38/GRCh38)
- SSS BAM inventory produced by sss_bam2.py
- Snakemake config.yaml with split_bam: True, cutoff (reads per single cell, default 10000; example 2000), and hybrid_filter (NC_007605,hs37d5)

## Outputs

- breakpointR_sce_summary.tsv with per-cell strand-state composition, SCE counts, pass/fail status, and chromosome coverage
- Per-cell relative chromosome coverage metrics
- Aggregate SCE counts per cell line (HAP1, Patski)

## How to apply

Collect the split single-cell BAM files from the split_bam step of the Snakemake pipeline (sciL3Pipe), using the SSS BAM inventory produced by sss_bam2.py after bwa alignment against hg38. Run breakpointR on each single-cell BAM with windowSize = 5e6, background = 0.05, and minReads = 50 to segment strand states and call sister chromatid exchanges per chromosome. Filter cells using the reported criteria: WC/uncategorized fraction <= 15% for haploid cells or <= 75% for diploid cells, and background between 0 and 0.08. Compute the pass rate among sorted cells (>58% expected) and aggregate per-cell SCE counts by cell line (expect 4 +/- 2.7 in HAP1 and 7 +/- 2.8 in Patski), then write per-cell strand-state composition, SCE counts, pass/fail status, and chromosome coverage to breakpointR_sce_summary.tsv. The rationale is that sci-L3-Strand-seq's nascent-strand nicking before IVT ensures only the unnicked template strand is amplified, so reciprocal strand-state switches across cells indicate SCEs.

## Related tools

- **breakpointR** (Segments strand states and calls sister chromatid exchanges per chromosome from single-cell Strand-seq BAMs (windowSize 5e6, background 0.05, minReads 50))
- **Snakemake** (Orchestrates the sciL3Pipe workflow including the split_bam step that produces per-cell BAMs; invoked via snakemake --configfile configs/config.yaml) — https://github.com/recombinationlab/sciL3Pipe
- **bwa** (Aligns sci-L3-Strand-seq reads to the hg38 (GRCh38) reference prior to single-cell BAM splitting) — https://github.com/recombinationlab/sciL3Pipe
- **sss_bam2.py** (Builds the SSS BAM inventory consumed downstream for splitting into single-cell BAMs) — https://github.com/recombinationlab/sciL3Pipe
- **Python** (Drives helper scripts (fastq2.py, trimmed2.py, sss_bam2.py) and downstream aggregation of per-cell SCE and QC metrics) — https://github.com/recombinationlab/sciL3Pipe

## Examples

```
python sss_bam2.py --bam_dir tests/bwa_alignment/
```

## Evaluation signals

- Reproduced pass rate exceeds 58% of sorted sci-L3-Strand-seq cells with strand information passing QC
- Mean SCE counts fall within reported ranges: 4 +/- 2.7 for HAP1 and 7 +/- 2.8 for Patski
- Filtered cells satisfy WC/uncategorized fraction <= 15% (haploid) or <= 75% (diploid) and background between 0 and 0.08
- Passing cells achieve an average of 0.22% genome coverage per cell
- breakpointR_sce_summary.tsv contains per-cell strand-state composition, SCE counts, pass/fail status, and chromosome coverage columns

## Limitations

- Adding additional rounds of cell barcoding under the sci framework reduces the number of recovered single cells due to loss of per-cell coverage
- Reads-per-cell cutoff governs cell retention (default 10000; example config 2000), affecting how many cells are available for SCE analysis
- No software version numbers are stated for the pipeline container (docker pulled automatically from dockerhub), complicating exact reproduction

## Evidence

- [methods] sss_bam2.py inventory step: [evidence span withheld — anchor status: not_found]
