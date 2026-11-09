---
name: genomic-interaction-distance-filtering
description: Use when comparing per-cell chromatin interaction counts across single-cell
  Hi-C protocols (sci-Hi-C, sci-L3-Hi-C, s3-GCC, droplet Hi-C) where raw valid-pair
  counts are inflated by linear-amplification (IVT) duplicates and short-distance
  interactions.
when_to_use_negative:
- Bulk Hi-C map construction where Juicer/cooler duplicate removal and filtering pipelines
  already apply the equivalent dedup and distance logic at the population level, not
  per cell
- Data lacking per-cell barcode segmentation (no cell identity per read pair), since
  the filter must collapse unique fragment pairs within each single cell
- Protocols with ligation junction enrichment where IVT duplicate inflation of unique
  fragment-pair counts is not a concern and the collapsing step would bias the comparison
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Python
  role: Downloads GSM2254217/GSE174226 data, parses valid-pairs/pairix records against
    hg38, applies the 1-kb distance filter, and collapses unique MboI fragment pairs
    per cell
- name: pairix
  role: Indexed contact-file format for the s3-GCC sci-L3-Hi-C records that are parsed
    per cell
- name: 4dn-dcic/docker-4dn-hic
  role: Containerized Hi-C processing (bam2pairs, pairsam filtering) that produces
    the pair-format interaction records this skill filters
  repo: https://github.com/4dn-dcic/docker-4dn-hic
- name: juicer
  role: Alternative Hi-C processing pipeline (with MboI restriction-site support)
    for generating deduplicated interaction lists from the same fastq inputs
  repo: https://github.com/aidenlab/juicer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:a79d017bc5593f74b1e353b4fca95efef6bef98d867d919041530d804b4b0cf2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/recombinationlab/sciL3Pipe
  repo_url: https://github.com/4dn-dcic/docker-4dn-hic
  tools:
  - 4dn-dcic/docker-4dn-hic
  - Python
  - juicer
  - pairix
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Genomic interaction distance filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A per-cell filtering and collapsing procedure for single-cell Hi-C data: apply a 1-kb interaction-distance filter to valid-pair records on hg38, then retain only interactions between unique MboI (GATC) restriction fragment pairs to remove IVT duplicate inflation, yielding comparable per-cell informative interaction counts across sci-Hi-C, sci-L3-Hi-C, and droplet Hi-C.

## When to use

Use when comparing per-cell chromatin interaction counts across single-cell Hi-C protocols (sci-Hi-C, sci-L3-Hi-C, s3-GCC, droplet Hi-C) where raw valid-pair counts are inflated by linear-amplification (IVT) duplicates and short-distance interactions.

## When NOT to use

- Bulk Hi-C map construction where Juicer/cooler duplicate removal and filtering pipelines already apply the equivalent dedup and distance logic at the population level, not per cell
- Data lacking per-cell barcode segmentation (no cell identity per read pair), since the filter must collapse unique fragment pairs within each single cell
- Protocols with ligation junction enrichment where IVT duplicate inflation of unique fragment-pair counts is not a concern and the collapsing step would bias the comparison

## Inputs

- sci-Hi-C valid-pairs file for GM12878 (ML3) cells (GSM2254217)
- s3-GCC pairix contact files for PDAC1/PDAC2 sci-L3-Hi-C cells (GSE174226)
- Droplet Hi-C per-cell interaction counts (Supplementary Table S2)
- hg38 (GRCh38) assembly chromosome coordinates
- MboI (GATC) restriction fragment definitions on hg38

## Outputs

- hic_comparison_medians.csv with per-cell interaction medians before/after filtering (sci-Hi-C: 1547→1390; sci-L3-Hi-C: 19385→7549)
- Hi-C-specific per-cell read estimates (total reads × 0.95 mapping rate × 5.8% or 6.3%)
- Per-cell counts of unique MboI fragment-pair interactions after the 1-kb distance filter

## How to apply

Parse per-cell valid-pairs/pairix records against the hg38 (GRCh38) assembly coordinates from sources such as GSM2254217 (sci-Hi-C ML3 GM12878) and GSE174226 (PDAC1/PDAC2 sci-L3-Hi-C). First compute per-cell interaction counts before filtering (sci-Hi-C expected median 1547; sci-L3-Hi-C expected 19385). Apply the 1-kb interaction-distance filter, then collapse contacts whose two read ends map to the same unique MboI (GATC) restriction fragment pair within each cell — this step is essential because IVT-based linear amplification duplicates molecules, inflating raw interaction counts. Recompute per-cell medians after filtering (expected 1390 for sci-Hi-C, 7549 for high-depth sci-L3-Hi-C) and benchmark against droplet Hi-C values in Supplementary Table S2. Complement the comparison with Hi-C-specific read estimates: total reads × 0.95 mapping rate × 5.8% for the 96-cell sci-Hi-C run and × 6.3% for the 929-cell sci-L3-Hi-C run.

## Related tools

- **Python** (Downloads GSM2254217/GSE174226 data, parses valid-pairs/pairix records against hg38, applies the 1-kb distance filter, and collapses unique MboI fragment pairs per cell)
- **pairix** (Indexed contact-file format for the s3-GCC sci-L3-Hi-C records that are parsed per cell)
- **4dn-dcic/docker-4dn-hic** (Containerized Hi-C processing (bam2pairs, pairsam filtering) that produces the pair-format interaction records this skill filters) — https://github.com/4dn-dcic/docker-4dn-hic
- **juicer** (Alternative Hi-C processing pipeline (with MboI restriction-site support) for generating deduplicated interaction lists from the same fastq inputs) — https://github.com/aidenlab/juicer

## Examples

```
pairs = pairs[(pairs.pos2 - pairs.pos1).abs() > 1000]; uniq = pairs.groupby(['cell','frag1','frag2']).size(); med = uniq.groupby(level=0).size().median()
```

## Evaluation signals

- Reproduced medians match published values: sci-Hi-C GM12878 1547 → 1390 unique fragment pairs per cell; sci-L3-Hi-C 19385 → 7549
- Post-filter per-cell counts never exceed pre-filter counts for the same cell
- All retained interactions span > 1 kb between the two mapped ends, and no two retained contacts share the same unique MboI fragment pair within a cell
- Hi-C-specific read estimates land at ~0.95 × 5.8% (sci-Hi-C, 96 cells) and ~0.95 × 6.3% (sci-L3-Hi-C, 929 cells) of total reads
- Output file hic_comparison_medians.csv contains both medians and Hi-C-specific read estimates for direct comparison against droplet Hi-C Supplementary Table S2 values

## Limitations

- sci-L3-Hi-C uses less frequent cutters (MboI) than other single-cell Hi-C methods that use cutters more frequent by 2-4x, so it captures fewer cis interactions
- IVT linear amplification inflates interaction-pair counts; counts that are not collapsed to unique MboI fragment pairs (1547, 19385) overstate library complexity
- Per-cell read cutoff must be chosen (pipeline default 10000; example config uses 2000); cell-cutoff criteria strongly affect which cells enter the median calculation

## Evidence

- [discussion] Restriction-enzyme cutter frequency limits cis interaction capture, a known confounder when comparing interaction counts across methods: [evidence span withheld — anchor status: not_found]
