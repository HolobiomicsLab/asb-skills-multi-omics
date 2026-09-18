---
name: restriction-fragment-annotation-mboi
description: Use when sci-L3-Hi-C pairs have been filtered to UU/UR/RU valid pairs
  and need restriction-fragment (MboI GATC) annotation and short-distance (<= 1 kb)
  interaction removal prior to downstream binning or QC statistics.
when_to_use_negative:
- Pairs that have not yet been marked as duplicates and filtered to UU/UR/RU categories
  — annotating unfiltered pairs will include invalid ligation products and duplicates
- Hi-C data digested with a restriction enzyme other than MboI (GATC) — fragment coordinates
  must match the assay's cutter
- Already-fragment-annotated or cooler/hic-format matrices where distance filtering
  has been applied at binning time
edam_topics:
- http://edamontology.org/topic_0157
tools:
- name: pairtools (restrict)
  role: Annotates retained UU/UR/RU valid pairs with MboI (GATC) restriction-fragment
    positions via the 4dn-dcic docker-4dn-hic wrapper
  repo: https://github.com/4dn-dcic/docker-4dn-hic
- name: Snakemake
  role: Orchestrates the sciL3Pipe workflow steps (markasdup, filter, restrict, distance
    filtering, stats) from configs/config.yaml
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: Docker
  role: Runs the docker-4dn-hic container (duplexa/4dn-hic:v43) that wraps the 4DN
    pairtools toolchain; automatically pulled from dockerhub
  repo: https://github.com/4dn-dcic/docker-4dn-hic
- name: bwa
  role: Upstream aligner producing the single-cell BAM files that feed the valid-pair
    filtering and restriction-fragment annotation stage
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:d632c743f75305a83ecb23e595984a91c719cfeae3f5a9ef6c77c7e3a20528d8
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
  - Apptainer
  - Docker
  - Python
  - Snakemake
  - bwa
  - pairtools (restrict)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# restriction-fragment-annotation-mboi

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate retained sci-L3-Hi-C valid interaction pairs with MboI (GATC) restriction-fragment positions using the pairtools restrict wrapper from the 4dn-dcic/docker-4dn-hic toolchain, then remove interaction pairs at genomic distances <= 1 kb before computing per-depth-class valid-pair statistics.

## When to use

Use when sci-L3-Hi-C pairs have been filtered to UU/UR/RU valid pairs and need restriction-fragment (MboI GATC) annotation and short-distance (<= 1 kb) interaction removal prior to downstream binning or QC statistics.

## When NOT to use

- Pairs that have not yet been marked as duplicates and filtered to UU/UR/RU categories — annotating unfiltered pairs will include invalid ligation products and duplicates
- Hi-C data digested with a restriction enzyme other than MboI (GATC) — fragment coordinates must match the assay's cutter
- Already-fragment-annotated or cooler/hic-format matrices where distance filtering has been applied at binning time

## Inputs

- Filtered valid-pair file containing UU, UR, and RU pair categories from pairtools markasdup/filter
- Barcode-annotated, bwa-aligned single-cell BAM files from the sciL3Pipe alignment module
- hg38 (GRCh38) reference genome assembly and MboI (GATC) restriction-fragment coordinates

## Outputs

- Restriction-fragment-annotated valid-pair file with MboI fragment positions per end
- Distance-filtered valid-pair set (pairs at genomic distance <= 1 kb removed)
- pairtools_valid_pair_stats.tsv with retention, duplicate-rate, and invalid-pair fractions

## How to apply

Start from the barcode-annotated, bwa-aligned single-cell BAM files enumerated by the sciL3Pipe manifest (sss_bam2.py --bam_dir tests/bwa_alignment/ style JSON), processed through the Snakemake workflow (snakemake --configfile configs/config.yaml) inside the automatically pulled docker-4dn-hic container (duplexa/4dn-hic:v43; use apptainer if Docker is unavailable). After pairtools markasdup labels duplicates and pairsam filter retains only UU, UR, and RU pairs, run the pairtools restrict step to assign each pair end to its overlapping MboI (GATC) fragment. Then drop interaction pairs at genomic distances <= 1 kb, since near-fragment self-ligations carry little conformational information. Aggregate the surviving pairs into the per-depth-class summary (pairtools_valid_pair_stats.tsv) and check retention against the reported 90.8% (low-pass) and 68.8% (high-depth) figures.

## Related tools

- **pairtools (restrict)** (Annotates retained UU/UR/RU valid pairs with MboI (GATC) restriction-fragment positions via the 4dn-dcic docker-4dn-hic wrapper) — https://github.com/4dn-dcic/docker-4dn-hic
- **Snakemake** (Orchestrates the sciL3Pipe workflow steps (markasdup, filter, restrict, distance filtering, stats) from configs/config.yaml) — https://github.com/recombinationlab/sciL3Pipe
- **Docker** (Runs the docker-4dn-hic container (duplexa/4dn-hic:v43) that wraps the 4DN pairtools toolchain; automatically pulled from dockerhub) — https://github.com/4dn-dcic/docker-4dn-hic
- **bwa** (Upstream aligner producing the single-cell BAM files that feed the valid-pair filtering and restriction-fragment annotation stage)

## Examples

```
snakemake --configfile configs/config.yaml
```

## Evaluation signals

- pairtools_valid_pair_stats.tsv reports valid-pair retention of 90.8% for low-pass and 68.8% for high-depth cells
- Duplicate rates in the stats file are 7.6% (low-pass) and 29.8% (high-depth)
- Invalid-pair fraction (non-UU/UR/RU) is below 1.5% in both depth classes
- No retained pairs lie at genomic distances <= 1 kb after the distance filter
- Every retained pair end carries an assigned MboI (GATC) restriction-fragment coordinate on the hg38 assembly

## Limitations

- sci-L3-Hi-C uses the less frequent MboI cutter; other single-cell Hi-C methods without ligation junction enrichment use cutters more frequent by 2-4x and capture more cis interactions
- No software version numbers are stated for the dockerhub-pulled pairtools toolchain, complicating exact reproduction
- The pipeline expects Docker (or Apptainer, which must be installed separately) to pull the container automatically

## Evidence

- [full_text] Annotate retained pairs with MboI (GATC) restriction-fragment positions using pairtools restrict, then remove interaction pairs at genomic distances <= 1 kb.: [evidence span withheld — anchor status: not_found]
- [results] valid pair retention figures: [evidence span withheld — anchor status: not_found]
- [readme] docker-4dn-hic container: [evidence span withheld — anchor status: not_found]
- [discussion] MboI cutter frequency limitation: [evidence span withheld — anchor status: not_found]
