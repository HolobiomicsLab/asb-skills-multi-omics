---
name: duplicate-read-pair-marking-and-classification
description: Use when processing barcode-annotated, bwa-aligned single-cell Hi-C pairsam/pair
  files and you need duplicate marking (markasdup), valid-pair class filtering (UU,
  UR, RU), and per-depth-class duplicate/retention statistics.
when_to_use_negative:
- Input is still raw FASTQ or untrimmed reads — use the upstream sciL3Pipe fastq2.py
  / trimmed2.py stages instead
- The goal is bulk Hi-C processing with Juicer — the aidenlab/juicer pipeline (juicer.sh
  with HindIII/MboI site) is a separate toolchain, not this sciL3Pipe/pairtools path
- 'Input BAMs have not been split into single-cell BAMs (split_bam: True step not
  yet run)'
edam_operation: http://edamontology.org/operation_3695
tools:
- name: Snakemake
  role: Orchestrates the sciL3Pipe workflow that wraps the 4dn pairtools markasdup/filter/restrict
    steps
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: Docker
  role: Runs the automatically pulled docker-4dn-hic container (duplexa/4dn-hic:v43)
    containing the pairtools toolchain and 4DN wrappers
  repo: https://github.com/4dn-dcic/docker-4dn-hic
- name: Apptainer
  role: Alternative container runtime for the docker-4dn-hic image if Docker is unavailable
  repo: https://github.com/4dn-dcic/docker-4dn-hic
- name: Python
  role: Used for the sss_bam2.py --bam_dir manifest step that enumerates the aligned
    single-cell BAM inputs
  repo: https://github.com/recombinationlab/sciL3Pipe
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:5e5d4fe200a8f12496709a56607da91322fef5e1b6a8b0863e6a9faa61ca1caa
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.1
n_claims: 10
---

# duplicate-read-pair-marking-and-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Marks duplicate read pairs and classifies filtered pair classes in sci-L3-Hi-C processing using the 4DN pairtools wrappers (run-pairsam-markasdup.sh, run-pairsam-filter.sh) from the docker-4dn-hic image duplexa/4dn-hic:v43, orchestrated by the sciL3Pipe Snakemake workflow.

## When to use

Use when processing barcode-annotated, bwa-aligned single-cell Hi-C pairsam/pair files and you need duplicate marking (markasdup), valid-pair class filtering (UU, UR, RU), and per-depth-class duplicate/retention statistics.

## When NOT to use

- Input is still raw FASTQ or untrimmed reads — use the upstream sciL3Pipe fastq2.py / trimmed2.py stages instead
- The goal is bulk Hi-C processing with Juicer — the aidenlab/juicer pipeline (juicer.sh with HindIII/MboI site) is a separate toolchain, not this sciL3Pipe/pairtools path
- Input BAMs have not been split into single-cell BAMs (split_bam: True step not yet run)

## Inputs

- barcode-annotated, bwa-aligned single-cell BAM files from the sciL3Pipe bwa_alignment stage (set enumerated via a sss_bam2.py --bam_dir style JSON manifest)
- sample/config YAML (configs/config.yaml) defining output_dir, temp_dir, fastqs, and alignment parameters
- genome assembly hg38 (GRCh38) with NC_007605,hs37d5 hybrid filter reference

## Outputs

- duplicate-marked pairsam files (pairtools markasdup)
- filtered valid-pair files restricted to UU, UR, and RU classes with MboI (GATC) fragment annotation
- pairtools_valid_pair_stats.tsv with per-depth-class valid-pair retention, duplicate rate, and invalid-pair fraction

## How to apply

Run the sciL3Pipe Snakemake workflow (snakemake --configfile configs/config.yaml) inside the automatically pulled docker-4dn-hic container (docker, or apptainer if installed) that wraps the 4dn pairtools toolchain, taking the barcode-annotated bwa-aligned single-cell BAMs from the upstream sciL3Pipe barcode/alignment module (enumerated via the sss_bam2.py --bam_dir tests/bwa_alignment/ style JSON manifest). Execute the 4DN wrapper run-pairsam-markasdup.sh to mark duplicate read pairs with pairtools markasdup, then run-pairsam-filter.sh to retain only UU, UR, and RU (unique/unwalked) pairs and discard invalid ligation products. Annotate retained pairs with MboI (GATC) restriction-fragment positions via pairtools restrict and remove interaction pairs at genomic distances <= 1 kb. Compute per-depth-class summary statistics and write them to pairtools_valid_pair_stats.tsv; expected values are 90.8% (low-pass) and 68.8% (high-depth) valid-pair retention, 7.6% and 29.8% duplicate rates, and <1.5% invalid pairs.

## Related tools

- **Snakemake** (Orchestrates the sciL3Pipe workflow that wraps the 4dn pairtools markasdup/filter/restrict steps) — https://github.com/recombinationlab/sciL3Pipe
- **Docker** (Runs the automatically pulled docker-4dn-hic container (duplexa/4dn-hic:v43) containing the pairtools toolchain and 4DN wrappers) — https://github.com/4dn-dcic/docker-4dn-hic
- **Apptainer** (Alternative container runtime for the docker-4dn-hic image if Docker is unavailable) — https://github.com/4dn-dcic/docker-4dn-hic
- **Python** (Used for the sss_bam2.py --bam_dir manifest step that enumerates the aligned single-cell BAM inputs) — https://github.com/recombinationlab/sciL3Pipe

## Examples

```
snakemake --configfile configs/config.yaml
```

## Evaluation signals

- pairtools_valid_pair_stats.tsv reports valid-pair retention of 90.8% for low-pass and 68.8% for high-depth cells
- Duplicate rates computed as 7.6% (low-pass) and 29.8% (high-depth)
- Invalid-pair fraction is below 1.5% after UU/UR/RU filtering
- Retained pairs carry MboI (GATC) restriction-fragment annotations and pairs at genomic distance <= 1 kb have been removed
- Input BAM set matches the sss_bam2.py --bam_dir tests/bwa_alignment/ JSON manifest enumeration

## Limitations

- No software version numbers are stated for the pairtools toolchain inside the docker image; the image tag (duplexa/4dn-hic:v43) is the only versioning anchor
- sci-L3-Hi-C uses MboI, a less frequent cutter than in other single-cell Hi-C methods (whose cutters are more frequent by 2-4x), capturing fewer cis interactions
- The docker container is automatically pulled from dockerhub, so results depend on the upstream image rather than pinned local tool versions

## Evidence

- [methods] Retained pairs are annotated with MboI (GATC) restriction-fragment positions using pairtools restrict, then interaction pairs at genomic distances <= 1 kb are removed: "The pipeline uses docker container which is automatically pulled from dockerhub. Note, apptainer needs to be installed."
- [methods] Valid-pair retention and duplicate-rate metrics anchor evaluation of the filtering stage: [evidence span withheld — anchor status: not_found]
- [readme] The pairtools toolchain is executed inside the 4dn-dcic docker-4dn-hic image, whose wrappers include run-pairsam-markasdup.sh and run-pairsam-filter.sh: [evidence span withheld — anchor status: not_found]
