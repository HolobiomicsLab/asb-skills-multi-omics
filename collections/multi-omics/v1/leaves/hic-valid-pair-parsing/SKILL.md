---
name: hic-valid-pair-parsing
description: Use when comparing per-cell informative chromatin interaction counts
  across single-cell Hi-C protocols from valid-pairs (e.g. GSM2254217) or pairix contact
  files, where duplicate IVT amplification products must be removed by collapsing
  to unique restriction fragment pairs.
when_to_use_negative:
- Input is bulk Hi-C data or aggregated contact matrices (e.g. .hic/.cool files) without
  per-cell barcode information — per-cell medians cannot be computed
- The comparison protocol uses a different restriction enzyme than MboI/GATC — fragment-pair
  collapsing keys would not match the published MboI-based deduplication logic
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3957
- http://edamontology.org/topic_0622
tools:
- name: Python
  role: Scripting language used to download GEO/SRA files, parse valid-pairs/pairix
    records, apply the 1-kb distance filter, collapse MboI fragment pairs per cell,
    and compute per-cell medians
- name: 4dn-dcic/docker-4dn-hic
  role: Containerized Hi-C processing pipeline (bam2pairs, pairix, cooler) whose pair-format
    conventions the valid-pair parsing follows
  repo: https://github.com/4dn-dcic/docker-4dn-hic
- name: aidenlab/juicer
  role: Alternative Hi-C processing platform for generating Hi-C maps from fastq data
    and .hic files with per-cell contact annotation
  repo: https://github.com/aidenlab/juicer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:4122a6b6b1e391f6c5513f6d652c59f5b7f55bd7462e77c3dbec80aa9dc9fc15
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
  - aidenlab/juicer
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3957
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# hic-valid-pair-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parse per-cell Hi-C valid-pair/pairix contact records against the hg38 (GRCh38) assembly, apply a 1-kb interaction-distance filter, and collapse contacts by unique MboI (GATC) restriction fragment pairs to compare per-cell informative interaction counts across single-cell Hi-C methods (sci-Hi-C vs sci-L3-Hi-C vs droplet Hi-C).

## When to use

Use when comparing per-cell informative chromatin interaction counts across single-cell Hi-C protocols from valid-pairs (e.g. GSM2254217) or pairix contact files, where duplicate IVT amplification products must be removed by collapsing to unique restriction fragment pairs.

## When NOT to use

- Input is bulk Hi-C data or aggregated contact matrices (e.g. .hic/.cool files) without per-cell barcode information — per-cell medians cannot be computed
- The comparison protocol uses a different restriction enzyme than MboI/GATC — fragment-pair collapsing keys would not match the published MboI-based deduplication logic

## Inputs

- sci-Hi-C ML3 GM12878 valid-pairs file (GEO GSM2254217)
- s3-GCC pairix contact files for PDAC1/PDAC2 sci-L3-Hi-C cells (GEO GSE174226)
- droplet Hi-C per-cell interaction counts (Supplementary Table S2)
- hg38 (GRCh38) genome assembly coordinates and MboI (GATC) restriction fragment map

## Outputs

- Per-cell unique MboI fragment pair counts after 1-kb distance filtering
- hic_comparison_medians.csv with reproduced medians (sci-Hi-C 1390, sci-L3-Hi-C 7549) and Hi-C-specific read estimates

## How to apply

Download the sci-Hi-C ML3 GM12878 valid-pairs file (GSM2254217), the s3-GCC pairix contact files for PDAC1/PDAC2 sci-L3-Hi-C cells (GSE174226), and the droplet Hi-C per-cell interaction counts (Supplementary Table S2). Parse valid-pairs/pairix records per cell against hg38 (GRCh38) coordinates, then apply the 1-kb interaction-distance filter. Collapse contacts whose two read ends map to the same unique MboI (GATC) restriction fragment pair within each cell — this prevents inflating counts with IVT-derived duplicates that share identical fragment combinations. Compute per-cell interaction medians before and after filtering: expected 1547 to 1390 for sci-Hi-C GM12878 and 19385 to 7549 for high-depth sci-L3-Hi-C cells. Optionally estimate Hi-C-specific reads per cell as total reads x 0.95 mapping rate x 5.8% (96-cell sci-Hi-C run) or x 6.3% (929-cell sci-L3-Hi-C run), and write results to hic_comparison_medians.csv.

## Related tools

- **Python** (Scripting language used to download GEO/SRA files, parse valid-pairs/pairix records, apply the 1-kb distance filter, collapse MboI fragment pairs per cell, and compute per-cell medians)
- **4dn-dcic/docker-4dn-hic** (Containerized Hi-C processing pipeline (bam2pairs, pairix, cooler) whose pair-format conventions the valid-pair parsing follows) — https://github.com/4dn-dcic/docker-4dn-hic
- **aidenlab/juicer** (Alternative Hi-C processing platform for generating Hi-C maps from fastq data and .hic files with per-cell contact annotation) — https://github.com/aidenlab/juicer

## Examples

```
python parse_valid_pairs.py --valid-pairs GSM2254217_valid_pairs.txt.gz --pairix GSE174226_pairs --assembly hg38 --min-dist 1000 --enzyme MboI --out hic_comparison_medians.csv
```

## Evaluation signals

- sci-Hi-C GM12878 per-cell median of interaction pairs drops from 1547 (pre-filter) to 1390 unique MboI fragment pairs (post-filter)
- High-depth sci-L3-Hi-C per-cell median drops from 19385 to 7549 unique fragment pairs after IVT-duplicate collapsing
- Every retained contact satisfies the 1-kb interaction-distance filter on hg38 coordinates
- Counts are computed only between unique combinations of MboI fragments per cell — re-running the collapse step yields identical per-cell medians (deterministic deduplication)
- Reproduced medians are consistent with the droplet Hi-C values in Supplementary Table S2

## Limitations

- sci-L3-Hi-C omits the ligation junction enrichment step, so counts include IVT duplicate interactions that must be collapsed by unique MboI fragment pairs; failing to do so inflates sci-L3-Hi-C counts substantially (19385 vs 7549)
- sci-L3-Hi-C uses less frequent cutters than other single-cell Hi-C methods (which use cutters more frequent by 2-4x), capturing fewer cis interactions
- Per-cell Hi-C-specific read fractions (5.8% for sci-Hi-C, 6.3% for sci-L3-Hi-C) are estimates derived from a 0.95 mapping rate and may vary by library

## Evidence

- [discussion] Limitation: less frequent cutters capture fewer cis interactions: [evidence span withheld — anchor status: not_found]
