---
name: valid-pair-retention-statistics-reporting
description: Use when barcode-annotated, bwa-aligned single-cell sci-L3-Hi-C BAMs
  have been processed through the pairtools markasdup/filter/restrict chain and per-depth-class
  retention, duplicate, and invalid-pair fractions must be computed and written to
  a stats table.
when_to_use_negative:
- Input is raw FASTQ rather than barcode-annotated, bwa-aligned single-cell BAMs —
  the upstream fastq2.py/trimmed2.py and bwa alignment stages must run first
- The assay is sci-L3-Strand-seq or sci-L3-RNA/ATAC rather than sci-L3-Hi-C — there
  are no ligation pairs to mark as duplicates or filter to UU/UR/RU classes
- A Hi-C workflow with ligation junction enrichment is used (e.g. Juicer-based) —
  this skill reflects the sci-L3-Hi-C design that omits ligation junction enrichment
  to yield combined WGS and interaction data per single cell
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3957
tools:
- name: Snakemake
  role: Drives the sciL3Pipe workflow that wraps the 4DN pairtools toolchain; invoked
    as snakemake --configfile configs/config.yaml
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: Docker
  role: Runs the automatically pulled dockerhub container containing the 4dn-dcic/docker-4dn-hic
    pairtools wrappers used for markasdup, filter, and restrict
  repo: https://github.com/4dn-dcic/docker-4dn-hic
- name: Apptainer
  role: Alternative container runtime for the 4dn-hic pairtools toolchain when Docker
    is not installed
  repo: https://github.com/4dn-dcic/docker-4dn-hic
- name: Python
  role: Implements the manifest and statistics steps, e.g. sss_bam2.py for enumerating
    the BAM inputs and computing/validating the per-depth-class summary statistics
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: sss_bam2.py
  role: Enumerates the single-cell BAM set (e.g. --bam_dir tests/bwa_alignment/) that
    feeds the pairtools valid-pair stage
  repo: https://github.com/recombinationlab/sciL3Pipe
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:81e8e6573e5111d02b701bbf3da8fdf2cefffbaec0cfd838897469efed12b4fe
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
  - Apptainer
  - Docker
  - Python
  - Snakemake
  - sss_bam2.py
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3957
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# valid-pair-retention-statistics-reporting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes and reports per-depth-class summary statistics for sci-L3-Hi-C valid chromatin interaction pairs after 4DN pairtools-based duplicate marking, valid-pair filtering, MboI fragment restriction, and short-distance removal, quantifying what fraction of aligned reads are retained as valid pairs.

## When to use

Use when barcode-annotated, bwa-aligned single-cell sci-L3-Hi-C BAMs have been processed through the pairtools markasdup/filter/restrict chain and per-depth-class retention, duplicate, and invalid-pair fractions must be computed and written to a stats table.

## When NOT to use

- Input is raw FASTQ rather than barcode-annotated, bwa-aligned single-cell BAMs — the upstream fastq2.py/trimmed2.py and bwa alignment stages must run first
- The assay is sci-L3-Strand-seq or sci-L3-RNA/ATAC rather than sci-L3-Hi-C — there are no ligation pairs to mark as duplicates or filter to UU/UR/RU classes
- A Hi-C workflow with ligation junction enrichment is used (e.g. Juicer-based) — this skill reflects the sci-L3-Hi-C design that omits ligation junction enrichment to yield combined WGS and interaction data per single cell

## Inputs

- barcode-annotated, bwa-aligned single-cell BAM files from the sciL3Pipe barcode/alignment module
- JSON manifest of BAM files (e.g. produced by sss_bam2.py --bam_dir tests/bwa_alignment/)
- sciL3Pipe Snakemake config (configs/config.yaml) with genome assembly hg38 and hybrid_filter NC_007605,hs37d5
- MboI (GATC) restriction-fragment definition for pairtools restrict

## Outputs

- pairtools_valid_pair_stats.tsv with per-depth-class valid-pair retention, duplicate rate, and invalid-pair fraction
- Filtered valid pair files restricted to UU/UR/RU pairs with MboI fragment annotations and pairs <= 1 kb removed
- Duplicate-marked pairsam files from pairtools markasdup

## How to apply

Take the barcode-annotated, bwa-aligned single-cell BAM files (enumerated via the sss_bam2.py --bam_dir tests/bwa_alignment/ style JSON manifest) and run the sciL3Pipe Snakemake workflow (snakemake --configfile configs/config.yaml) inside the automatically pulled dockerhub container that wraps the 4dn-dcic/docker-4dn-hic pairtools toolchain (Apptainer if Docker is unavailable). Mark duplicate read pairs with pairtools markasdup, then filter the pair file to keep only UU, UR, and RU (unique/unwalked) pairs, discarding invalid ligation products. Annotate retained pairs with MboI (GATC) restriction-fragment positions via pairtools restrict and remove interaction pairs at genomic distances <= 1 kb. Finally compute per-depth-class summary statistics — valid-pair retention of 90.8% for low-pass and 68.8% for high-depth cells, duplicate rates of 7.6% (low-pass) and 29.8% (high-depth), and invalid-pair fractions below 1.5% — and write them to pairtools_valid_pair_stats.tsv, verifying the computed values match the reported figures.

## Related tools

- **Snakemake** (Drives the sciL3Pipe workflow that wraps the 4DN pairtools toolchain; invoked as snakemake --configfile configs/config.yaml) — https://github.com/recombinationlab/sciL3Pipe
- **Docker** (Runs the automatically pulled dockerhub container containing the 4dn-dcic/docker-4dn-hic pairtools wrappers used for markasdup, filter, and restrict) — https://github.com/4dn-dcic/docker-4dn-hic
- **Apptainer** (Alternative container runtime for the 4dn-hic pairtools toolchain when Docker is not installed) — https://github.com/4dn-dcic/docker-4dn-hic
- **Python** (Implements the manifest and statistics steps, e.g. sss_bam2.py for enumerating the BAM inputs and computing/validating the per-depth-class summary statistics) — https://github.com/recombinationlab/sciL3Pipe
- **sss_bam2.py** (Enumerates the single-cell BAM set (e.g. --bam_dir tests/bwa_alignment/) that feeds the pairtools valid-pair stage) — https://github.com/recombinationlab/sciL3Pipe

## Examples

```
snakemake --configfile configs/config.yaml
```

## Evaluation signals

- pairtools_valid_pair_stats.tsv reports valid-pair retention of 90.8% for low-pass and 68.8% for high-depth aligned reads
- Duplicate rates match the reported 7.6% (low-pass) and 29.8% (high-depth) values
- Invalid-pair fraction is below 1.5% in both depth classes
- Filtered pair file contains only UU, UR, and RU pair types with walk/failed-ligation classes discarded
- All retained pairs carry MboI (GATC) fragment annotations and pairs at genomic distances <= 1 kb are absent

## Limitations

- sci-L3-Hi-C uses less frequent cutters than other single-cell Hi-C methods (which use cutters more frequent by 2-4x), capturing fewer cis interactions
- Omitting the ligation junction enrichment step yields a combination of WGS and interaction (Hi-C) data per cell, so retention statistics reflect a mixed WGS/Hi-C read population
- No software version numbers are stated for the pipeline's docker container, complicating exact reproduction of the pairtools behavior

## Evidence

- [results] After filtering, sci-L3-Hi-C retained 90.8% of initial low-pass aligned reads and 68.8% of high-depth aligned reads, with 7.6% and 29.8% duplicates respectively and <1.5% invalid pairs.: [evidence span withheld — anchor status: not_found]
- [methods] workflow step 4: [evidence span withheld — anchor status: not_found]
- [methods] workflow step 5: [evidence span withheld — anchor status: not_found]
- [methods] workflow step 3: [evidence span withheld — anchor status: not_found]
