---
name: snakemake-workflow-execution
description: Use when raw sci-L3 FASTQ directories or barcode-annotated BAM inventories
  need to be processed end-to-end via the sciL3Pipe Snakemake workflow with configurable
  cutoffs, hybrid filters, and containerized tools.
when_to_use_negative:
- Data is already aligned and split into single-cell BAMs and only a downstream analysis
  like breakpointR strand-state calling is needed — skip the FASTQ-to-BAM workflow
  stages
- The workflow is a non-sci-L3 Hi-C dataset better served by the full Juicer or ENCODE
  hic-pipeline rather than the sciL3Pipe pairtools wrappers
- No Snakemake (>= 9.11.6) and no container runtime (Docker or Apptainer) is installable
  in the target environment
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Snakemake
  role: Workflow engine executing the sciL3Pipe DAG from FASTQ preprocessing through
    split_bam; requires >= 9.11.6 and --configfile configs/config.yaml
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: Docker
  role: Container runtime whose image is automatically pulled from dockerhub to wrap
    the 4dn pairtools toolchain
  repo: https://github.com/4dn-dcic/docker-4dn-hic
- name: Apptainer
  role: Alternative container runtime used instead of Docker when Docker is not installed
- name: bwa
  role: 'Genome aligner configured via aligner: ''bwa'' to align sci-L3 reads to hg38
    with hybrid filtering against NC_007605/hs37d5'
- name: fastq2.py / fastq2.py
  role: Creates the sample JSON manifest linking R1/R2 FASTQ files per sample for
    the pipeline
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: sss_bam2.py
  role: Builds BAM inventories from aligned BAM directories (e.g. tests/bwa_alignment/)
    feeding split_bam
  repo: https://github.com/recombinationlab/sciL3Pipe
- name: snakemake-executor-plugin-cluster-hoffman2
  role: Cluster executor plugin for submitting Snakemake jobs to Hoffman2-style cluster
    schedulers
  repo: https://github.com/peterch405/snakemake-executor-plugin-cluster-hoffman2
merged_aliases:
- containerized-snakemake-workflow-execution
merged_alias_records:
- alias: containerized-snakemake-workflow-execution
  slug: containerized-snakemake-workflow-execution
  jaccard_score: 0.75
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:c7e22a3232a0524e7fb61597517a3240745bdf7cc656ca005e54ec524340df95
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
  - breakpointR
  - bwa
  - fastq2.py / fastq2.py
  - snakemake-executor-plugin-cluster-hoffman2
  - sss_bam2.py
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.2
n_claims: 10
---

# snakemake-workflow-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs the sciL3Pipe Snakemake pipeline (minimum snakemake 9.11.6) inside an automatically pulled Docker/Apptainer container to take sci-L3 FASTQs from trimming through bwa alignment, split_bam single-cell BAM generation, and 4DN pairtools Hi-C processing with a YAML config defining paths, mismatch tolerances, and filter references.

## When to use

Use when raw sci-L3 FASTQ directories or barcode-annotated BAM inventories need to be processed end-to-end via the sciL3Pipe Snakemake workflow with configurable cutoffs, hybrid filters, and containerized tools.

## When NOT to use

- Data is already aligned and split into single-cell BAMs and only a downstream analysis like breakpointR strand-state calling is needed — skip the FASTQ-to-BAM workflow stages
- The workflow is a non-sci-L3 Hi-C dataset better served by the full Juicer or ENCODE hic-pipeline rather than the sciL3Pipe pairtools wrappers
- No Snakemake (>= 9.11.6) and no container runtime (Docker or Apptainer) is installable in the target environment

## Inputs

- Directory of raw gzipped FASTQ files (paired R1/R2), optionally symbolically linked
- sample. manifest mapping sample IDs to R1/R2 FASTQ paths produced by fastq2.py
- configs/config.yaml with output_dir, temp_dir, aligner, assembly, mismatch settings, cutoff, hybrid_filter, and split_bam flags
- Barcode-annotated, bwa-aligned single-cell BAM directory (enumerable via sss_bam2.py --bam_dir manifest) for split_bam

## Outputs

- Trimmed FASTQs, bwa-aligned BAMs, and split single-cell BAMs under output_dir (e.g. tests/) and temp_dir
- Downstream pairtools outputs (markedasdup pairsam, filtered UU/UR/RU valid pairs, restricted fragment annotations) when the Hi-C container toolchain runs
- Per-sample statistics tables such as pairtools_valid_pair_stats.tsv and breakpointR_sce_summary.tsv built on the split BAMs

## How to apply

Create a folder of symbolic links to raw FASTQs, then generate the sample manifest JSON with `python fastq2.py --fastq_dir tests/fastqs/` (or `fastq2.py`/`trimmed2.py`/`sss_bam2.py` for downstream manifest creation from trimmed FASTQs or BAM directories). Author a config.yaml that sets output_dir, temp_dir, aligner 'bwa', assembly 'hg38', hybrid_filter 'NC_007605,hs37d5', additional_filter 'hs37d5', per-barcode mismatch tolerances (sss_mismatches: 0, rt_mismatches: 3, tn5_mismatches: 1, ligation_mismatches: 1), a per-cell read cutoff (cutoff: 2000), and split_bam: True to split SSS BAMs into single-cell BAMs. Launch the workflow with `snakemake --configfile configs/config.yaml`; the pipeline automatically pulls the docker container from dockerhub (4dn-dcic/docker-4dn-hic for the pairtools toolchain), or uses apptainer if Docker is unavailable. For cluster execution, install snakemake-executor-plugin-cluster-generic plus the hoffman2-specific executor plugin. Decision points include choosing process_only ('human') for hybrid assemblies and verifying the config against the included tests/config.yaml.

## Related tools

- **Snakemake** (Workflow engine executing the sciL3Pipe DAG from FASTQ preprocessing through split_bam; requires >= 9.11.6 and --configfile configs/config.yaml) — https://github.com/recombinationlab/sciL3Pipe
- **Docker** (Container runtime whose image is automatically pulled from dockerhub to wrap the 4dn pairtools toolchain) — https://github.com/4dn-dcic/docker-4dn-hic
- **Apptainer** (Alternative container runtime used instead of Docker when Docker is not installed)
- **bwa** (Genome aligner configured via aligner: 'bwa' to align sci-L3 reads to hg38 with hybrid filtering against NC_007605/hs37d5)
- **fastq2.py / fastq2.py** (Creates the sample JSON manifest linking R1/R2 FASTQ files per sample for the pipeline) — https://github.com/recombinationlab/sciL3Pipe
- **sss_bam2.py** (Builds BAM inventories from aligned BAM directories (e.g. tests/bwa_alignment/) feeding split_bam) — https://github.com/recombinationlab/sciL3Pipe
- **snakemake-executor-plugin-cluster-hoffman2** (Cluster executor plugin for submitting Snakemake jobs to Hoffman2-style cluster schedulers) — https://github.com/peterch405/snakemake-executor-plugin-cluster-hoffman2

## Examples

```
snakemake --configfile configs/config.yaml
```

## Evaluation signals

- The config.yaml parses and matches the schema of the included tests/config.yaml example (output_dir, temp_dir, aligner, assembly, mismatch keys, cutoff, hybrid_filter present)
- snakemake 9.11.6+ is installed along with snakemake-executor-plugin-cluster-generic before launch
- The container is automatically pulled from dockerhub (or apptainer runs it) with no manual tool installation needed for the pairtools toolchain
- split_bam: True produces one single-cell BAM per cell from each SSS BAM, with cells below the cutoff: 2000 reads dropped
- Downstream statistics reconcile with reported figures: Strand-seq QC pass rate > 58% with 0.22% mean per-cell genome coverage, and Hi-C valid-pair retention of 90.8% (low-pass) / 68.8% (high-depth) with < 1.5% invalid pairs

## Limitations

- No software version numbers are stated beyond the minimum snakemake 9.11.6; the docker image tag (duplexa/4dn-hic:v43) may change
- Apptainer needs to be explicitly installed if Docker is unavailable; no fallback workflow exists without a container runtime
- The repo README has no changelog, so exact tool versions inside the container are not traceable from the repository alone
- sci-L3-Hi-C uses less frequent cutters (2-4x less frequent than other single-cell Hi-C methods), capturing fewer cis interactions, so pairtools valid-pair outputs will show relatively fewer cis pairs than MboI-frequency protocols

## Evidence

- [methods] sss_bam2.py BAM manifest creation: [evidence span withheld — anchor status: not_found]
