---
name: single-cell-hic-pairtools-pair-filtering
description: Use when processing single-cell Hi-C (e.g. sci-L3-Hi-C) aligned pairs
  into valid interaction pairs, needing duplicate marking, UU/UR/RU pair selection,
  MboI fragment annotation, and valid-pair retention/duplicate-rate statistics.
when_to_use_negative:
- Input is still raw FASTQ — the upstream sciL3Pipe barcode/trim/alignment module
  (fastq2.py, trimmed2.py, bwa) must run first
- Data is bulk (non-single-cell) Hi-C or uses a different ligation-junction-enriched
  protocol — sci-L3-Hi-C specifically omits ligation junction enrichment and uses
  MboI, a less frequent cutter than many single-cell Hi-C methods
- Genome assembly is not hg38/GRCh38 and the hybrid_filter references (NC_007605,
  hs37d5) are not applicable
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0106
tools:
- name: Snakemake
  role: Orchestrates the pairtools-based pair filtering workflow via configs/config.yaml
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: 4dn-dcic/docker-4dn-hic
  role: Container image wrapping the 4DN pairtools toolchain including run-pairsam-markasdup.sh
    and run-pairsam-filter.sh wrappers
  repo: https://github.com/4dn-dcic/docker-4dn-hic
- name: bwa
  role: Produces the aligned single-cell BAMs that are the input to pair filtering
- name: sss_bam2.py
  role: Enumerates the split single-cell BAM directory fed into the Hi-C pair stage
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: Apptainer
  role: Alternative container runtime if docker is unavailable
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:11eb9e0c42c49396d614fa75a49ae3c6ae83b3b14c3149518886c93274afca5b
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
  - 4dn-dcic/docker-4dn-hic
  - Apptainer
  - Docker
  - Python
  - Snakemake
  - bwa
  - sss_bam2.py
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0106
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-hic-pairtools-pair-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the sci-L3-Hi-C valid-pair derivation stage: single-cell barcode-annotated, bwa-aligned BAMs are converted to pairsam format and processed with the 4DN pairtools wrappers (markasdup, filter, restrict) to yield deduplicated, restriction-fragment-annotated cis pairs above 1 kb, with per-depth-class retention statistics.

## When to use

Use when processing single-cell Hi-C (e.g. sci-L3-Hi-C) aligned pairs into valid interaction pairs, needing duplicate marking, UU/UR/RU pair selection, MboI fragment annotation, and valid-pair retention/duplicate-rate statistics.

## When NOT to use

- Input is still raw FASTQ — the upstream sciL3Pipe barcode/trim/alignment module (fastq2.py, trimmed2.py, bwa) must run first
- Data is bulk (non-single-cell) Hi-C or uses a different ligation-junction-enriched protocol — sci-L3-Hi-C specifically omits ligation junction enrichment and uses MboI, a less frequent cutter than many single-cell Hi-C methods
- Genome assembly is not hg38/GRCh38 and the hybrid_filter references (NC_007605, hs37d5) are not applicable

## Inputs

- Barcode-annotated, bwa-aligned single-cell BAM files from the sciL3Pipe barcode/alignment module (hg38/GRCh38 assembly)
- Sample manifest JSON enumerating per-cell BAM paths (sss_bam2.py --bam_dir style)
- sciL3Pipe configs/config.yaml with hybrid_filter "NC_007605,hs37d5" and additional_filter "hs37d5"

## Outputs

- Deduplicated, filtered pairsam files containing UU/UR/RU valid pairs with MboI restriction-fragment annotations
- pairtools_valid_pair_stats.tsv with per-depth-class valid-pair retention, duplicate rates, and invalid-pair fractions
- Distance-filtered (<= 1 kb removed) interaction pairs suitable for downstream cooler/juicer processing

## How to apply

Start from the barcode-annotated, bwa-aligned single-cell BAM files enumerated by the sss_bam2.py manifest step of sciL3Pipe. Run the sciL3Pipe Snakemake workflow (snakemake --configfile configs/config.yaml) inside the automatically pulled 4dn-dcic/docker-4dn-hic container (docker, or apptainer if installed), which wraps the pairtools toolchain. Mark duplicate read pairs with pairtools markasdup, then filter the pairs file to retain only UU, UR, and RU (unique/unwalked) pairs, discarding invalid ligation products. Annotate retained pairs with MboI (GATC) restriction-fragment positions via pairtools restrict and remove interaction pairs at genomic distances <= 1 kb. Finally, compute per-depth-class statistics (valid-pair retention, duplicate rates, invalid-pair fractions) and write them to pairtools_valid_pair_stats.tsv, checking them against the reported 90.8%/68.8% retention, 7.6%/29.8% duplicates, and <1.5% invalid pairs.

## Related tools

- **Snakemake** (Orchestrates the pairtools-based pair filtering workflow via configs/config.yaml) — https://github.com/recombinationlab/sciL3Pipe
- **4dn-dcic/docker-4dn-hic** (Container image wrapping the 4DN pairtools toolchain including run-pairsam-markasdup.sh and run-pairsam-filter.sh wrappers) — https://github.com/4dn-dcic/docker-4dn-hic
- **bwa** (Produces the aligned single-cell BAMs that are the input to pair filtering)
- **sss_bam2.py** (Enumerates the split single-cell BAM directory fed into the Hi-C pair stage) — https://github.com/recombinationlab/sciL3Pipe
- **Apptainer** (Alternative container runtime if docker is unavailable)

## Examples

```
snakemake --configfile configs/config.yaml
```

## Evaluation signals

- pairtools_valid_pair_stats.tsv reproduces reported figures: 90.8% valid-pair retention for low-pass and 68.8% for high-depth aligned reads
- Duplicate rates fall at 7.6% (low-pass) and 29.8% (high-depth), and invalid-pair fractions are below 1.5%
- Filtered pairs file contains only UU, UR, and RU pair types; all other ligation products discarded
- Every retained pair carries MboI (GATC) restriction-fragment annotations, and no pairs with genomic distance <= 1 kb remain
- Pipeline runs inside the duplexa/4dn-hic (docker-4dn-hic) container without manual pairtools installation

## Limitations

- sci-L3-Hi-C uses MboI, a less frequent cutter (by 2-4x) than those of other single-cell Hi-C methods without ligation junction enrichment, capturing fewer cis interactions
- No software version numbers are stated for the pairtools toolchain (docker image is pulled automatically from dockerhub)
- High-depth cells lose substantially more reads to duplicates (29.8% vs 7.6% for low-pass), so retention depends strongly on sequencing depth class

## Evidence

- [readme] Container image source providing the pairtools wrappers used in this stage: [evidence span withheld — anchor status: not_found]
- [discussion] Cutter-frequency caveat limiting cis-interaction capture for this MboI-based protocol: [evidence span withheld — anchor status: not_found]
