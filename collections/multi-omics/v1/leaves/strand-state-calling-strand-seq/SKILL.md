---
name: strand-state-calling-strand-seq
description: Use when processing sci-L3-Strand-seq libraries to call per-cell strand
  states, detect sister chromatid exchanges (SCEs), apply the paper's cell QC criteria,
  and reproduce reported pass rates and SCE counts in HAP1/Patski cell lines.
when_to_use_negative:
- Input data are sci-L3-Hi-C or sci-L3-RNA/ATAC libraries — strand-state calling and
  SCE detection only apply to the Strand-seq modality, which requires nascent strand
  nicking before IVT
- You have unsplit pooled SSS BAMs — the skill requires per-cell BAMs produced by
  the split_bam step, so split them first with sss_bam2.py/split_bam before QC
- Your goal is Hi-C contact maps or genome compartmentalization (A/B compartments)
  rather than template-strand states and SCEs — use the Hi-C-specific pipeline components
  instead
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: breakpointR
  role: Segments strand states (windowSize = 5e6, background = 0.05, minReads = 50)
    and calls sister chromatid exchanges per chromosome from single-cell Strand-seq
    BAMs
- name: Snakemake
  role: Orchestrates the sciL3Pipe workflow including the split_bam step that produces
    per-cell BAMs for strand-state calling
  repo: github.com/recombinationlab/sciL3Pipe
- name: sss_bam2.py
  role: Builds the SSS BAM inventory that is the entry point for collecting single-cell
    BAMs before QC
  repo: github.com/recombinationlab/sciL3Pipe
- name: bwa
  role: Aligner used upstream to align sci-L3 reads to hg38 before BAM splitting and
    strand-state calling
  repo: github.com/recombinationlab/sciL3Pipe
- name: Python
  role: Used for aggregation scripts that compute pass rates, SCE counts by cell line,
    and chromosome coverage, writing breakpointR_sce_summary.tsv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:6a2a324bc6784aecc155c676ab92adaff4d7613421829e203fbccf5faa13f245
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# strand-state-calling-strand-seq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A QC and analysis skill for sci-L3-Strand-seq single-cell data that runs breakpointR strand-state segmentation on split single-cell BAMs, filters cells by WC/uncategorized fraction and background, and reports pass rates, per-cell SCE counts, and chromosome coverage.

## When to use

Use when processing sci-L3-Strand-seq libraries to call per-cell strand states, detect sister chromatid exchanges (SCEs), apply the paper's cell QC criteria, and reproduce reported pass rates and SCE counts in HAP1/Patski cell lines.

## When NOT to use

- Input data are sci-L3-Hi-C or sci-L3-RNA/ATAC libraries — strand-state calling and SCE detection only apply to the Strand-seq modality, which requires nascent strand nicking before IVT
- You have unsplit pooled SSS BAMs — the skill requires per-cell BAMs produced by the split_bam step, so split them first with sss_bam2.py/split_bam before QC
- Your goal is Hi-C contact maps or genome compartmentalization (A/B compartments) rather than template-strand states and SCEs — use the Hi-C-specific pipeline components instead

## Inputs

- Split single-cell BAM files from the split_bam step of the Snakemake pipeline
- SSS BAM inventory produced by sss_bam2.py
- hg38 (GRCh38) reference assembly
- Snakemake config.yaml with cutoff: 2000 reads per single cell and split_bam: True

## Outputs

- breakpointR_sce_summary.tsv with per-cell strand-state composition, SCE counts, pass/fail QC status, and chromosome coverage
- Per-cell QC pass rate (fraction of sorted cells passing strand-state filters)
- Per-cell-line mean SCE counts (HAP1, Patski)
- Relative chromosome coverage per cell

## How to apply

First collect the split single-cell BAMs produced by the split_bam step of the sciL3Pipe Snakemake pipeline (after inventorying SSS BAMs with sss_bam2.py). Run breakpointR on each single-cell BAM with windowSize = 5e6, background = 0.05, and minReads = 50 to segment strand states and call sister chromatid exchanges per chromosome. Filter cells using the reported criteria: WC/uncategorized fraction <= 15% for haploid cells or <= 75% for diploid cells, and background between 0 and 0.08. Then compute the pass rate among all sorted cells (expected >58%), aggregate per-cell SCE counts by cell line (expect 4 +/- 2.7 in HAP1 and 7 +/- 2.8 in Patski), and compute relative chromosome coverage per cell. Write per-cell strand-state composition, SCE counts, pass/fail status, and chromosome coverage to breakpointR_sce_summary.tsv.

## Related tools

- **breakpointR** (Segments strand states (windowSize = 5e6, background = 0.05, minReads = 50) and calls sister chromatid exchanges per chromosome from single-cell Strand-seq BAMs)
- **Snakemake** (Orchestrates the sciL3Pipe workflow including the split_bam step that produces per-cell BAMs for strand-state calling) — github.com/recombinationlab/sciL3Pipe
- **sss_bam2.py** (Builds the SSS BAM inventory that is the entry point for collecting single-cell BAMs before QC) — github.com/recombinationlab/sciL3Pipe
- **bwa** (Aligner used upstream to align sci-L3 reads to hg38 before BAM splitting and strand-state calling) — github.com/recombinationlab/sciL3Pipe
- **Python** (Used for aggregation scripts that compute pass rates, SCE counts by cell line, and chromosome coverage, writing breakpointR_sce_summary.tsv)

## Examples

```
python sss_bam2.py --bam_dir tests/bwa_alignment/; snakemake --configfile configs/config.yaml --use-conda
```

## Evaluation signals

- Reproduced QC pass rate exceeds 58% of sorted sci-L3-Strand-seq cells with strand information
- Mean SCE counts fall within reported ranges: 4 +/- 2.7 in HAP1 and 7 +/- 2.8 in Patski
- Average relative genome coverage per passing cell is approximately 0.22%
- Every passing cell satisfies WC/uncategorized fraction <= 15% (haploid) or <= 75% (diploid) and background in [0, 0.08]
- breakpointR_sce_summary.tsv contains strand-state composition, SCE counts, pass/fail status, and chromosome coverage for every cell with strand information

## Limitations

- Additional rounds of cell barcoding reduce the number of recovered single cells relative to cells sorted, due to loss of per-cell coverage
- Cell sorting must enrich for BrdU-labeled cells of the right cell cycle; cells lacking strand information cannot be QC'd
- No software version numbers are stated for the pipeline tools, complicating exact reproduction
- The pipeline (sciL3Pipe) requires snakemake >= 9.11.6 and container runtimes (Docker or Apptainer), which may not be available in all environments

## Evidence

- [methods] workflow uses breakpointR with reported QC criteria: [evidence span withheld — anchor status: not_found]
