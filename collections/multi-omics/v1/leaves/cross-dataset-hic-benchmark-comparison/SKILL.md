---
name: cross-dataset-hic-benchmark-comparison
description: Use when benchmarking per-cell informative chromatin interaction counts
  across single-cell Hi-C modalities, or when harmonizing published valid-pair datasets
  (e.g. GSM2254217, GSE174226) under identical distance and deduplication filters.
when_to_use_negative:
- Raw fastq-level processing or alignment of sci-L3 data is needed — use the sciL3Pipe
  Snakemake pipeline instead of this benchmark-comparison skill
- The comparison target is contact-map/compartment analysis rather than per-cell library
  complexity counting
- Data are already deduplicated at the fragment-pair level, making the IVT-duplicate
  collapse step unnecessary and the pre/post-filter medians redundant
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3922
- http://edamontology.org/topic_0654
tools:
- name: Python
  role: Download of GSM2254217/GSE174226 records, parsing of valid-pairs/pairix files
    per cell, MboI fragment-pair collapsing, and median computation
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: Snakemake
  role: Upstream pipeline orchestration for regenerating sci-L3 processed data when
    raw fastqs must be reprocessed before comparison
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: bowtie2
  role: Alignment against the hg38 (GRCh38) bowtie2 index when valid-pair records
    must be regenerated from reads
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: Juicer
  role: Alternative Hi-C valid-pair generation and deduplication tooling referenced
    for standard Hi-C processing conventions
  repo: https://github.com/aidenlab/juicer
- name: docker-4dn-hic
  role: Containerized pairix/pairsam tooling for producing pairix contact files comparable
    to the GSE174226 inputs
  repo: https://github.com/4dn-dcic/docker-4dn-hic
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:dddd2c1f73f14637680ed8225ac5b22da0412298824447829bd4375aed431443
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
  - Juicer
  - Python
  - Snakemake
  - bowtie2
  - docker-4dn-hic
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3922
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cross-dataset-hic-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the head-to-head comparison of single-cell Hi-C library complexity (sci-L3-Hi-C vs sci-Hi-C vs s3-GCC vs droplet Hi-C) by re-analyzing published valid-pair/contact data under a harmonized 1-kb interaction-distance filter, unique MboI fragment-pair collapsing, and per-cell cutoff criteria.

## When to use

Use when benchmarking per-cell informative chromatin interaction counts across single-cell Hi-C modalities, or when harmonizing published valid-pair datasets (e.g. GSM2254217, GSE174226) under identical distance and deduplication filters.

## When NOT to use

- Raw fastq-level processing or alignment of sci-L3 data is needed — use the sciL3Pipe Snakemake pipeline instead of this benchmark-comparison skill
- The comparison target is contact-map/compartment analysis rather than per-cell library complexity counting
- Data are already deduplicated at the fragment-pair level, making the IVT-duplicate collapse step unnecessary and the pre/post-filter medians redundant

## Inputs

- sci-Hi-C ML3 GM12878 valid-pairs file (GSM2254217)
- s3-GCC pairix contact files for PDAC1/PDAC2 sci-L3-Hi-C cells (GSE174226)
- Droplet Hi-C per-cell interaction counts (Supplementary Table S2)
- hg38 (GRCh38) genome assembly coordinates
- MboI (GATC) restriction-fragment coordinate map

## Outputs

- hic_comparison_medians.csv with per-cell interaction medians before/after filtering and Hi-C-specific read estimates
- Per-cell counts of unique MboI fragment-pair interactions (sci-Hi-C median 1390; sci-L3-Hi-C median 7549)

## How to apply

Download the sci-Hi-C ML3 GM12878 valid-pairs file (GSM2254217), the s3-GCC pairix contact files for PDAC1/PDAC2 sci-L3-Hi-C cells (GSE174226), and droplet Hi-C per-cell counts (Supplementary Table S2) with Python. Parse valid-pair/pairix records per cell against hg38 (GRCh38) coordinates, apply the 1-kb interaction-distance filter, then collapse contacts whose two read ends map to the same unique MboI (GATC) restriction fragment pair within each cell — this step is critical because IVT-based linear amplification in sci-L3 duplicates molecules, so raw interaction pairs inflate complexity. Compute per-cell medians before and after filtering (sci-Hi-C expected 1547 → 1390; sci-L3-Hi-C expected 19385 → 7549) and compare against droplet Hi-C values. Optionally estimate Hi-C-specific reads per cell as total reads × 0.95 mapping rate × 5.8% (96-cell sci-Hi-C run) or × 6.3% (929-cell sci-L3-Hi-C run), and write results to hic_comparison_medians.csv.

## Related tools

- **Python** (Download of GSM2254217/GSE174226 records, parsing of valid-pairs/pairix files per cell, MboI fragment-pair collapsing, and median computation) — https://github.com/recombinationlab/sciL3Pipe
- **Snakemake** (Upstream pipeline orchestration for regenerating sci-L3 processed data when raw fastqs must be reprocessed before comparison) — https://github.com/recombinationlab/sciL3Pipe
- **bowtie2** (Alignment against the hg38 (GRCh38) bowtie2 index when valid-pair records must be regenerated from reads) — https://github.com/recombinationlab/sciL3Pipe
- **Juicer** (Alternative Hi-C valid-pair generation and deduplication tooling referenced for standard Hi-C processing conventions) — https://github.com/aidenlab/juicer
- **docker-4dn-hic** (Containerized pairix/pairsam tooling for producing pairix contact files comparable to the GSE174226 inputs) — https://github.com/4dn-dcic/docker-4dn-hic

## Examples

```
python hic_comparison_medians.py --sci_hic_gsm GSM2254217 --scil3_gse GSE174226 --assembly hg38 --mindist 1000 --enzyme MboI --cutoff 2000 --out hic_comparison_medians.csv
```

## Evaluation signals

- sci-Hi-C GM12878 per-cell median of interaction pairs drops from 1547 to 1390 unique fragment pairs after MboI fragment-pair collapsing
- High-depth sci-L3-Hi-C per-cell median drops from 19385 to 7549 unique fragment pairs after collapsing
- sci-L3-Hi-C median (7549) remains higher than sci-Hi-C median (1390) post-filter, confirming greater library complexity despite omitted ligation junction enrichment
- Estimated Hi-C-specific reads per cell fall within expected magnitudes using 0.95 mapping rate × 5.8% (sci-Hi-C 96-cell run) and × 6.3% (sci-L3-Hi-C 929-cell run)
- All contacts pass the 1-kb interaction-distance filter before fragment-pair collapsing, and every contact maps to hg38 (GRCh38) coordinates

## Limitations

- sci-L3-Hi-C uses less frequent cutters than other single-cell Hi-C methods (which use cutters more frequent by 2-4x), capturing fewer cis interactions
- IVT linear amplification inflates raw interaction-pair counts (19385 vs 7549 post-filter), so raw medians are not directly comparable across modalities without fragment-pair collapsing
- Adding additional rounds of cell barcoding under the sci framework reduces the number of recovered single cells due to loss of per-cell coverage

## Evidence

- [results] After counting only interactions between unique combinations of MboI fragments (to avoid inflating counts with IVT duplicates), sci-Hi-C GM12878 cells have a median of 1390 unique fragment pairs per cell: [evidence span withheld — anchor status: not_found]
- [discussion] Other single-cell Hi-C methods without ligation junction enrichment utilize more frequent cutters by 2-4x, obtaining a higher capture of cis interactions: [evidence span withheld — anchor status: not_found]
- [intro] we omitted the ligation junction enrichment step conventionally used [11, 13], leading us to obtain a combination of whole genome sequencing (WGS) and interaction (Hi-C) data for each single cell: "we omitted the ligation junction enrichment step conventionally used [11, 13], leading us to obtain a combination of whole genome sequencing (WGS) and interaction (Hi-C) data for each single cell"
- [readme] A Snakemake pipeline for processing and aligning sci-L3-seq data.: "A Snakemake pipeline for processing and aligning sci-L3-seq data."
