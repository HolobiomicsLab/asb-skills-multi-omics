---
name: nextflow-pipeline-execution-with-containers
description: Use when a task requires launching a multi-step nf-core/DSL2 workflow
  such as eduomics, and reproducible, containerised execution (per-process containers)
  on a target compute infrastructure is needed.
when_to_use_negative:
- The analysis is a single ad-hoc step (e.g. running FastQC alone) rather than the
  full eduomics simulation-to-validation workflow — invoke the individual tool directly
  instead of launching the whole pipeline.
- You intend to pass pipeline parameters through a custom config file with `-c` —
  the README explicitly forbids parameters in custom config files; use the CLI or
  `-params-file`.
- The target environment has no container engine and no Conda fallback — per-process
  DSL2 containers require one of Docker, Singularity, or Conda.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Nextflow
  role: 'DSL2 workflow engine that orchestrates the eduomics simulation pipeline;
    requires version ≥25.04.4 per the article (README badge: ≥25.04.3)'
  repo: https://www.nextflow.io/
- name: Docker
  role: Container system supplied as a -profile option so each pipeline process runs
    in an isolated, reproducible container
  repo: https://www.docker.com/
- name: Singularity
  role: Alternative container system profile for HPC environments where Docker is
    unavailable
  repo: https://sylabs.io/docs/
- name: Conda
  role: Alternative environment-based profile when container runtimes are not permitted
  repo: https://docs.conda.io/en/latest/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:e935eb7d463c49b6fa9345ac195b7fc25ac57b081c60d7647d3ec2e8f97db4c1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Conda
  - Docker
  - FastQC
  - MultiQC
  - Nextflow
  - Singularity
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# nextflow-pipeline-execution-with-containers

## Summary

Runs the eduomics nf-core pipeline (DSL2) under Nextflow with a container engine profile (Docker, Singularity, or Conda) so that each process executes in an isolated, reproducible environment. This underpins reproducible generation of simulated DNA/RNA-seq teaching datasets with known ground truth.

## When to use

Use when a task requires launching a multi-step nf-core/DSL2 workflow such as eduomics, and reproducible, containerised execution (per-process containers) on a target compute infrastructure is needed.

## When NOT to use

- The analysis is a single ad-hoc step (e.g. running FastQC alone) rather than the full eduomics simulation-to-validation workflow — invoke the individual tool directly instead of launching the whole pipeline.
- You intend to pass pipeline parameters through a custom config file with `-c` — the README explicitly forbids parameters in custom config files; use the CLI or `-params-file`.
- The target environment has no container engine and no Conda fallback — per-process DSL2 containers require one of Docker, Singularity, or Conda.

## Inputs

- samplesheet.csv with columns id,type,chromosome,coverage,capture,reps,simthreshold (one row per DNA or RNA simulation)
- Reference genome configuration (recommended: GATK.GRCh38)
- Container engine profile choice (docker, singularity, or conda)

## Outputs

- Organised output directory (--outdir) containing simulated FASTQ files, reference bundles, validation results, and AI-generated educational scenarios
- Nextflow work directory, pipeline reports, and per-process containerised execution logs

## How to apply

Install Nextflow version ≥25.04.4 and choose one container system — Docker, Singularity, or Conda — passing it as the -profile argument (e.g. -profile docker). Prepare a CSV samplesheet where each row is one simulation with columns id, type (dna or rna), chromosome (e.g. chr22), coverage, capture (BED URL for DNA), reps, groups, and simthreshold (e.g. 0.3); run `nextflow run eduomics --input samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>`. The DSL2 implementation uses one container per process, many sourced from nf-core/modules, which keeps dependencies isolated per step (e.g. FastQC, MultiQC, SimuSCoP, Salmon, DESeq2) and makes results reproducible. Before running on real simulation parameters, test the setup with `-profile test` per nf-core conventions. Supply all pipeline parameters via the CLI or a `-params-file`; custom config files passed with `-c` may configure resources but must NOT carry parameters.

## Related tools

- **Nextflow** (DSL2 workflow engine that orchestrates the eduomics simulation pipeline; requires version ≥25.04.4 per the article (README badge: ≥25.04.3)) — https://www.nextflow.io/
- **Docker** (Container system supplied as a -profile option so each pipeline process runs in an isolated, reproducible container) — https://www.docker.com/
- **Singularity** (Alternative container system profile for HPC environments where Docker is unavailable) — https://sylabs.io/docs/
- **Conda** (Alternative environment-based profile when container runtimes are not permitted) — https://docs.conda.io/en/latest/

## Examples

```
nextflow run eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- A dry run with `-profile test` completes before real data is submitted (nf-core recommended setup check).
- Every process resolves its own container image (one container per process in DSL2) — a missing engine profile should fail fast at launch.
- The run completes for a samplesheet row with type=dna (chr22, coverage=100, simthreshold=0.3) or type=rna (chr22, coverage=30, reps=3, groups=2).
- All outputs appear under the --outdir path with the expected DNA/RNA simulation substructure (FASTQ, reference bundle, validation results, educational scenario).
- CI workflows (ci.yml, linting.yml) and nf-test unit tests pass on the lescai-teaching/eduomics repository.

## Limitations

- Parameters cannot be provided via custom `-c` config files; only via CLI or `-params-file`.
- Pipeline assumes chromosome-specific analysis (typically chr22) and capture-based (exome/targeted) DNA sequencing with paired-end reads against a human reference (GATK.GRCh38).
- Only simulations passing the built-in validation step are included in the final pipeline output, so failed simulations terminate without producing final packages.

## Evidence

- [intro] Nextflow and container engines: "**Nextflow installed** (version ≥25.04.4)"
- [intro] Container system options: "**Container system** (Docker, Singularity, or Conda)"
- [readme] DSL2 one-container-per-process design: [evidence span withheld — anchor status: not_found]
- [readme] Concrete run command: "nextflow run eduomics \
   -profile <docker/singularity/.../institute> \
   --input samplesheet.csv \
   --genome GATK.GRCh38 \
   --outdir <OUTDIR>"
- [readme] Parameters must not go in -c configs: [evidence span withheld — anchor status: not_found]
