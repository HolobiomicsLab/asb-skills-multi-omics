---
name: single-cell-hic-median-computation
description: Use when comparing per-cell interaction complexity across single-cell
  Hi-C protocols from published valid-pair/contact files, especially when IVT-amplified
  libraries (sci-L3) must be deduplicated against restriction-fragment combinations
  to avoid inflated counts.
when_to_use_negative:
- Input is aggregated bulk Hi-C contact matrices (e.g. .hic or .cool) with no per-cell
  barcode structure — per-cell medians cannot be computed
- The comparison aims at cis-interaction capture efficiency across enzymes with different
  cutting frequencies; note sci-L3-Hi-C uses less frequent cutters (2-4x) than other
  single-cell Hi-C methods, so raw cis counts are not directly comparable without
  accounting for cutter frequency
- Data are already deduplicated valid pairs restricted to unique fragment pairs —
  re-collapsing would not change the medians
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3957
- http://edamontology.org/topic_0622
tools:
- name: Python
  role: Download the GEO/SRA valid-pair and pairix files, parse per-cell records,
    apply the 1-kb filter, collapse to unique MboI fragment pairs, and compute per-cell
    medians
- name: Snakemake
  role: Orchestrates the sci-L3Pipe processing steps that produce per-cell BAMs/pairs
    upstream of the median computation
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: bwa
  role: Aligner used in the sci-L3 pipeline to map reads to hg38 before valid-pair
    extraction
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: bowtie2
  role: Alternative aligner configured against the GRCh38 bowtie2 index in the sci-L3
    pipeline
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: docker-4dn-hic
  role: Provides bam2pairs/pairix tooling for generating and querying per-cell contact
    files
  repo: https://github.com/4dn-dcic/docker-4dn-hic
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:66a45f9f196223e4023b9c3ac58f18b9c67abe3b2382b16f6f9c895dfd43a9b1
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
  - bowtie2
  - bwa
  - docker-4dn-hic
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3957
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# single-cell-hic-median-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces head-to-head comparisons of per-cell informative chromatin interaction counts across single-cell Hi-C modalities (sci-L3-Hi-C, sci-Hi-C, s3-GCC, droplet Hi-C) by parsing valid-pair records against hg38, applying a 1-kb interaction-distance filter, collapsing contacts to unique MboI (GATC) fragment pairs per cell, and reporting per-cell medians before and after filtering.

## When to use

Use when comparing per-cell interaction complexity across single-cell Hi-C protocols from published valid-pair/contact files, especially when IVT-amplified libraries (sci-L3) must be deduplicated against restriction-fragment combinations to avoid inflated counts.

## When NOT to use

- Input is aggregated bulk Hi-C contact matrices (e.g. .hic or .cool) with no per-cell barcode structure — per-cell medians cannot be computed
- The comparison aims at cis-interaction capture efficiency across enzymes with different cutting frequencies; note sci-L3-Hi-C uses less frequent cutters (2-4x) than other single-cell Hi-C methods, so raw cis counts are not directly comparable without accounting for cutter frequency
- Data are already deduplicated valid pairs restricted to unique fragment pairs — re-collapsing would not change the medians

## Inputs

- sci-Hi-C GM12878 (ML3) valid-pairs file (GSM2254217)
- s3-GCC pairix contact files for PDAC1/PDAC2 sci-L3-Hi-C cells (GSE174226)
- droplet Hi-C per-cell interaction counts (Supplementary Table S2)
- hg38 (GRCh38) assembly coordinates
- MboI (GATC) restriction fragment map of hg38

## Outputs

- Per-cell unique fragment-pair counts before and after the 1-kb distance filter and MboI-pair collapse
- hic_comparison_medians.csv with reproduced medians and Hi-C-specific read estimates
- Comparison table against droplet Hi-C Supplementary Table S2 values

## How to apply

Download the sci-Hi-C ML3 GM12878 valid-pairs file (GSM2254217), the s3-GCC pairix contact files for PDAC1/PDAC2 sci-L3-Hi-C cells (GSE174226), and the droplet Hi-C per-cell counts (Supplementary Table S2). Parse valid-pair/pairix records per cell against hg38 (GRCh38) coordinates, then apply the 1-kb interaction-distance filter. Collapse contacts whose two read ends map to the same unique MboI (GATC) restriction fragment pair within each cell — this is critical for sci-L3-Hi-C because linear IVT amplification inflates raw interaction counts with duplicates of the same fragment pair. Compute per-cell medians before and after filtering (sci-Hi-C expected to drop 1547 → 1390; sci-L3-Hi-C 19385 → 7549) and compare against droplet Hi-C values in Supplementary Table S2. Optionally estimate Hi-C-specific reads per cell as total reads × 0.95 mapping rate × 5.8% (96-cell sci-Hi-C run) or × 6.3% (929-cell sci-L3-Hi-C run), and write results to hic_comparison_medians.csv.

## Related tools

- **Python** (Download the GEO/SRA valid-pair and pairix files, parse per-cell records, apply the 1-kb filter, collapse to unique MboI fragment pairs, and compute per-cell medians)
- **Snakemake** (Orchestrates the sci-L3Pipe processing steps that produce per-cell BAMs/pairs upstream of the median computation) — https://github.com/recombinationlab/sciL3Pipe
- **bwa** (Aligner used in the sci-L3 pipeline to map reads to hg38 before valid-pair extraction) — https://github.com/recombinationlab/sciL3Pipe
- **bowtie2** (Alternative aligner configured against the GRCh38 bowtie2 index in the sci-L3 pipeline) — https://github.com/recombinationlab/sciL3Pipe
- **docker-4dn-hic** (Provides bam2pairs/pairix tooling for generating and querying per-cell contact files) — https://github.com/4dn-dcic/docker-4dn-hic

## Examples

```
python fastq2.py --fastq_dir tests/fastqs/  # then parse per-cell pairs, filter >1 kb, collapse to unique MboI (GATC) fragment pairs, and write hic_comparison_medians.csv
```

## Evaluation signals

- sci-Hi-C GM12878 per-cell median of interaction pairs is 1547 before filtering and drops to 1390 unique MboI fragment pairs after filtering
- sci-L3-Hi-C (PDAC1/PDAC2) per-cell median drops from 19385 interaction pairs to 7549 unique fragment pairs after MboI-pair collapse
- Hi-C-specific read estimates reconcile with total reads × 0.95 mapping rate × 5.8% (sci-Hi-C, 96 cells) and × 6.3% (sci-L3-Hi-C, 929 cells)
- sci-L3-Hi-C retains a higher per-cell median of unique fragment pairs (7549) than sci-Hi-C (1390) despite omitting ligation junction enrichment
- hic_comparison_medians.csv is written with medians matching the droplet Hi-C values cross-checked from Supplementary Table S2

## Limitations

- sci-L3-Hi-C uses restriction cutters less frequent by 2-4x than other single-cell Hi-C methods without ligation junction enrichment, capturing fewer cis interactions, so cross-protocol comparisons are sensitive to cutter choice
- Omitting the MboI unique-fragment-pair collapse inflates counts with IVT-derived duplicates, particularly for linearly amplified sci-L3 libraries (median drops ~2.6x from 19385 to 7549)
- No software version numbers are stated for the pipeline components, and the associated repository has no changelog, complicating exact reproduction

## Evidence

- [results] After counting only interactions between unique combinations of MboI fragments, sci-Hi-C GM12878 cells have a median of 1390 unique fragment pairs per cell (down from 1547 interaction pairs), while high-depth sci-L3-Hi-C cells have a median of 7549 (down from 19 385): [evidence span withheld — anchor status: not_found]
- [discussion] Other single-cell Hi-C methods use more frequent cutters, capturing more cis interactions: [evidence span withheld — anchor status: not_found]
