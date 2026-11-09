---
name: restriction-fragment-pair-deduplication
description: Use when computing per-cell informative chromatin interaction counts
  from single-cell Hi-C valid-pairs/pairix records and IVT-based amplification duplicates
  must be removed by collapsing contacts to unique restriction-fragment combinations
  before reporting medians.
when_to_use_negative:
- The valid-pairs records are already collapsed to unique restriction-fragment combinations
  — re-running the collapse will not change counts and wastes computation.
- The goal is cis-interaction capture efficiency across methods with different cutters
  rather than duplicate-aware per-cell interaction complexity counts.
- Input is bulk (population) Hi-C where duplicate removal is handled by standard deduplication
  in tools like Juicer rather than per-cell fragment-pair counting.
tools:
- name: Python
  role: Used to parse valid-pairs/pairix records per cell, apply the 1-kb distance
    filter, and collapse contacts sharing the same unique MboI fragment pair within
    each cell
- name: bwa
  role: Aligns sequencing reads to hg38 (GRCh38) so read ends can be assigned to restriction
    fragments
- name: sciL3Pipe
  role: Snakemake pipeline that processes sci-L3 fastq files into per-cell BAMs whose
    contacts can be fragment-pair deduplicated
  repo: https://github.com/recombinationlab/sciL3Pipe
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:2157c75ec35459b2d6b19cc5f9f5250a011ae2553479b2a04b624e471b2b3e88
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
  - bwa
  - sciL3Pipe
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# restriction-fragment-pair-deduplication

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A Hi-C counting correction that collapses per-cell interaction pairs whose two read ends map to the same unique MboI (GATC) restriction fragment pair, preventing IVT/PCR duplicate molecules from inflating single-cell chromatin interaction counts. Applied after a 1-kb interaction-distance filter, it reduced sci-Hi-C GM12878 medians from 1547 to 1390 unique fragment pairs per cell and sci-L3-Hi-C medians from 19385 to 7549.

## When to use

Use when computing per-cell informative chromatin interaction counts from single-cell Hi-C valid-pairs/pairix records and IVT-based amplification duplicates must be removed by collapsing contacts to unique restriction-fragment combinations before reporting medians.

## When NOT to use

- The valid-pairs records are already collapsed to unique restriction-fragment combinations — re-running the collapse will not change counts and wastes computation.
- The goal is cis-interaction capture efficiency across methods with different cutters rather than duplicate-aware per-cell interaction complexity counts.
- Input is bulk (population) Hi-C where duplicate removal is handled by standard deduplication in tools like Juicer rather than per-cell fragment-pair counting.

## Inputs

- sci-Hi-C GM12878 (ML3) valid-pairs file (GSM2254217)
- s3-GCC pairix contact files for PDAC1/PDAC2 sci-L3-Hi-C cells (GSE174226)
- hg38 (GRCh38) reference genome coordinates and MboI (GATC) restriction fragment map
- droplet Hi-C per-cell interaction counts (Supplementary Table S2) for benchmarking

## Outputs

- Per-cell counts of unique MboI restriction fragment pairs after the 1-kb distance filter
- Per-cell interaction medians before and after deduplication (e.g. 1547 -> 1390 for sci-Hi-C; 19385 -> 7549 for sci-L3-Hi-C)
- hic_comparison_medians.csv with reproduced medians and Hi-C-specific read estimates

## How to apply

Parse valid-pairs or pairix contact records per cell against the hg38 (GRCh38) assembly, keeping only interactions at distances greater than 1 kb. For each contact within a cell, map both read ends to their overlapping MboI (GATC) restriction fragments and collapse contacts whose two ends map to the same unique combination of MboI fragments, since IVT amplification produces duplicate copies of the same original molecule that would otherwise inflate interaction counts. Compute per-cell medians before and after collapsing (expected: sci-Hi-C GM12878 ML3 1547 -> 1390; high-depth sci-L3-Hi-C 19385 -> 7549) and apply the same cell-cutoff criteria (e.g. the sciL3Pipe default of 10000 reads per cell, set to 2000 in the example config) to both methods being compared. Note that sci-L3-Hi-C omits ligation junction enrichment, so Hi-C-specific reads per cell are estimated as total reads x 0.95 mapping rate x 5.8% (96-cell sci-Hi-C run) or 6.3% (929-cell sci-L3-Hi-C run) rather than from junction-enrichment statistics.

## Related tools

- **Python** (Used to parse valid-pairs/pairix records per cell, apply the 1-kb distance filter, and collapse contacts sharing the same unique MboI fragment pair within each cell)
- **bwa** (Aligns sequencing reads to hg38 (GRCh38) so read ends can be assigned to restriction fragments)
- **sciL3Pipe** (Snakemake pipeline that processes sci-L3 fastq files into per-cell BAMs whose contacts can be fragment-pair deduplicated) — https://github.com/recombinationlab/sciL3Pipe

## Examples

```
med = df[df.dist > 1000].groupby('cell').apply(lambda g: len(set(zip(g.frag1, g.frag2))))  # unique MboI fragment pairs per cell
```

## Evaluation signals

- After deduplication, sci-Hi-C GM12878 cells reproduce a median of 1390 unique fragment pairs per cell (down from 1547 interaction pairs).
- High-depth sci-L3-Hi-C cells reproduce a median of 7549 unique fragment pairs per cell (down from 19385), confirming more complex chromatin interaction libraries.
- No counted pair may involve non-unique (e.g. multi-mapping or ambiguous MboI fragment) fragment combinations within a cell.
- The 1-kb interaction-distance filter and the same per-cell read cutoff (sciL3Pipe example sets cutoff: 2000) must be applied identically to both methods before comparing medians.
- Reproduced medians and Hi-C-specific read estimates (total reads x 0.95 x 5.8% or 6.3%) are written to hic_comparison_medians.csv and benchmarked against droplet Hi-C Supplementary Table S2 values.

## Limitations

- sci-L3-Hi-C uses less frequent cutters than other single-cell Hi-C methods (which use cutters more frequent by 2-4x), capturing fewer cis interactions and yielding lower apparent interaction complexity.
- Without collapsing unique MboI fragment combinations, IVT-amplified duplicates of the same molecule inflate per-cell interaction counts (e.g. sci-L3-Hi-C median drops from 19385 to 7549 once deduplicated).
- The pipeline repo provides no changelog and no stated software version numbers, so exact reproducing environments may be hard to pin down.

## Evidence

- [discussion] Other single-cell Hi-C methods without ligation junction enrichment utilize more frequent cutters by 2-4x, obtaining a higher capture of cis interactions: [evidence span withheld — anchor status: not_found]
- [other] Apply the 1-kb interaction-distance filter, then collapse contacts whose two read ends map to the same unique MboI (GATC) restriction fragment pair within each cell.: [evidence span withheld — anchor status: not_found]
- [methods] # Reads per single cell to keep (default: 10000)
cutoff: 2000: "# Reads per single cell to keep (default: 10000)
cutoff: 2000"
