---
name: chromosome-coverage-normalization
description: Use when processing split single-cell BAMs from a sci-L3-Strand-seq run
  and you need per-cell relative chromosome coverage, strand-state composition, and
  SCE counts for QC against reported pass-rate thresholds.
when_to_use_negative:
- Input is pooled or library-level BAM data rather than split single-cell BAMs — per-cell
  strand states and coverage cannot be computed.
- The goal is Hi-C contact-map generation or RNA/ATAC quantification from sci-L3 co-assays;
  those modalities use different downstream pipelines (juicer/cooler-based Hi-C processing,
  RNA read recovery) rather than breakpointR strand-state segmentation.
- Cells have fewer than the configured reads-per-single-cell cutoff (default 10000,
  config default 2000 in the example config); coverage estimates will be too sparse
  to normalize meaningfully.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: breakpointR
  role: Segments strand states and calls sister chromatid exchanges per chromosome
    on each single-cell BAM using windowSize=5e6, background=0.05, minReads=50; its
    output feeds the per-cell chromosome coverage computation
- name: Snakemake
  role: Drives the sciL3Pipe workflow including the split_bam step that produces the
    single-cell BAMs this skill consumes
  repo: github.com/recombinationlab/sciL3Pipe
- name: sss_bam2.py
  role: Loads the SSS BAM inventory from the bwa_alignment directory to enumerate
    the single-cell BAMs prior to splitting and breakpointR analysis
  repo: github.com/recombinationlab/sciL3Pipe
- name: bwa
  role: Aligner configured in the sciL3Pipe config that generated the SSS BAMs upstream
    of the split single-cell BAMs
  repo: github.com/recombinationlab/sciL3Pipe
- name: Python
  role: Used for the per-cell aggregation of strand-state composition, SCE counts,
    and relative chromosome coverage into breakpointR_sce_summary.tsv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:39d8809d925ac605408942af21e51c5311e16cfc2da8cd44ca3bb9709b71c250
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
  - breakpointR
  - bwa
  - sss_bam2.py
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# chromosome-coverage-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute relative per-chromosome coverage for sci-L3-Strand-seq single cells after breakpointR strand-state segmentation, so that strand-state composition, sister chromatid exchange (SCE) counts, and genome coverage per cell can be compared across HAP1 and Patski cell lines against the paper's QC benchmarks.

## When to use

Use when processing split single-cell BAMs from a sci-L3-Strand-seq run and you need per-cell relative chromosome coverage, strand-state composition, and SCE counts for QC against reported pass-rate thresholds.

## When NOT to use

- Input is pooled or library-level BAM data rather than split single-cell BAMs — per-cell strand states and coverage cannot be computed.
- The goal is Hi-C contact-map generation or RNA/ATAC quantification from sci-L3 co-assays; those modalities use different downstream pipelines (juicer/cooler-based Hi-C processing, RNA read recovery) rather than breakpointR strand-state segmentation.
- Cells have fewer than the configured reads-per-single-cell cutoff (default 10000, config default 2000 in the example config); coverage estimates will be too sparse to normalize meaningfully.

## Inputs

- Split single-cell BAM files from the split_bam step of the sciL3Pipe Snakemake pipeline
- SSS BAM inventory produced by sss_bam2.py
- Cell line labels (haploid HAP1 vs diploid Patski) for applying the correct WC/uncategorized fraction threshold

## Outputs

- breakpointR_sce_summary.tsv with per-cell strand-state composition, SCE counts, pass/fail status, and relative chromosome coverage
- Per-cell pass rate over sorted cells for QC comparison against the reported >58%
- Mean SCE counts aggregated by cell line (expected 4 +/- 2.7 in HAP1 and 7 +/- 2.8 in Patski)

## How to apply

Take the split single-cell BAMs produced by the split_bam step of the sciL3Pipe Snakemake pipeline (fed by sss_bam2.py on the SSS BAM inventory), and run breakpointR on each cell-level BAM with windowSize = 5e6, background = 0.05, and minReads = 50 to segment strand states and call sister chromatid exchanges per chromosome. Apply the paper's cell filters: WC/uncategorized fraction <= 15% for haploid cells (HAP1) or <= 75% for diploid cells (Patski), and background between 0 and 0.08. For each cell that passes, compute relative coverage per chromosome from the segmented genome and aggregate per-cell SCE counts by cell line. Write the per-cell strand-state composition, SCE counts, pass/fail status, and chromosome coverage to breakpointR_sce_summary.tsv. The rationale is that linear-amplification (IVT-based) sci-L3 libraries yield low per-cell coverage (average 0.22% of the genome per cell), so coverage must be normalized per chromosome to interpret strand states reliably.

## Related tools

- **breakpointR** (Segments strand states and calls sister chromatid exchanges per chromosome on each single-cell BAM using windowSize=5e6, background=0.05, minReads=50; its output feeds the per-cell chromosome coverage computation)
- **Snakemake** (Drives the sciL3Pipe workflow including the split_bam step that produces the single-cell BAMs this skill consumes) — github.com/recombinationlab/sciL3Pipe
- **sss_bam2.py** (Loads the SSS BAM inventory from the bwa_alignment directory to enumerate the single-cell BAMs prior to splitting and breakpointR analysis) — github.com/recombinationlab/sciL3Pipe
- **bwa** (Aligner configured in the sciL3Pipe config that generated the SSS BAMs upstream of the split single-cell BAMs) — github.com/recombinationlab/sciL3Pipe
- **Python** (Used for the per-cell aggregation of strand-state composition, SCE counts, and relative chromosome coverage into breakpointR_sce_summary.tsv)

## Examples

```
python sss_bam2.py --bam_dir tests/bwa_alignment/
```

## Evaluation signals

- Reproduced pass rate of sorted sci-L3-Strand-seq cells after filtering exceeds the reported >58% of sorted cells with strand information
- Mean per-cell SCE counts fall within the reported ranges: 4 +/- 2.7 for HAP1 and 7 +/- 2.8 for Patski
- Filtered cells satisfy WC/uncategorized fraction <= 15% (haploid) or <= 75% (diploid) and background between 0 and 0.08
- Per-cell relative chromosome coverage values are consistent with the reported average of 0.22% genome coverage per cell
- breakpointR_sce_summary.tsv contains strand-state composition, SCE counts, pass/fail status, and chromosome coverage for every sorted cell

## Limitations

- Adding additional rounds of cell barcoding under the sci framework reduces the number of recovered single cells and per-cell coverage, which shrinks the set of cells usable for coverage normalization.
- Low RNA read recovery is a known limitation of the sci-L3 co-assay; modalities other than Strand-seq are not the target of this skill.
- No software version numbers are stated in the methods (Docker/Apptainer containers are pulled from dockerhub), so breakpointR and pipeline versions must be pinned independently for reproducibility.
- No changelog was found for the associated repository, complicating provenance tracking of pipeline changes.

## Evidence

- [methods] Run breakpointR on each single-cell BAM with windowSize = 5e6, background = 0.05, and minReads = 50: [evidence span withheld — anchor status: not_found]
- [methods] quality criteria for strand-state QC: [evidence span withheld — anchor status: not_found]
- [results] SCE counts by cell line: [evidence span withheld — anchor status: not_found]
- [abstract] stranded template amplification enables SCE detection: "we show our ability to detect sister chromatid exchanges, genome compartmentalization, and cell state-specific features in thousands of single cells."
- [readme] pipeline overview from README: "# perform splitting of SSS BAMs into single cell BAMs
split_bam: True"
- [readme] reads-per-cell cutoff in pipeline config: "# Reads per single cell to keep (default: 10000)
cutoff: 2000"
