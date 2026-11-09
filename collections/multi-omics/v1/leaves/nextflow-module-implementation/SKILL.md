---
name: nextflow-module-implementation
description: Use when a new analytical step must be added to a Nextflow/nf-core pipeline
  as an isolated, containerised process with defined inputs, channels, and outputs
  (e.g. a variant-extraction module in eduomics).
when_to_use_negative:
- The step runs outside any workflow manager and a standalone script is sufficient
- You only need to run an existing published pipeline — use `nextflow run nf-core/eduomics`
  directly rather than implementing a new module
- The target pipeline does not follow the nf-core DSL2 one-container-per-process structure
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Nextflow
  role: Workflow framework in which the module is implemented as a DSL2 process (version
    >=25.04)
  repo: https://github.com/lescai-teaching/eduomics
- name: Docker
  role: Container engine used to execute the module process reproducibly
  repo: https://github.com/lescai-teaching/eduomics
- name: Singularity
  role: Alternative container engine for the module process
  repo: https://github.com/lescai-teaching/eduomics
- name: Conda
  role: Alternative non-container execution profile for the module process
  repo: https://github.com/lescai-teaching/eduomics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:ca44fc4665e8cc52d74d4e4a41372863dbd9fdaf18408741127e73756ae11367
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lescai-teaching/eduomics
  tools:
  - Conda
  - Docker
  - Nextflow
  - Singularity
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Nextflow module implementation

## Summary

Implement a pipeline component (e.g. the SUBVAR pathogenic-variant extraction module in eduomics) as a Nextflow DSL2 process, executed via Docker, Singularity, or Conda, with tool-specific parameters (e.g. GATK.GRCh38 reference) supplied through pipeline configuration.

## When to use

Use when a new analytical step must be added to a Nextflow/nf-core pipeline as an isolated, containerised process with defined inputs, channels, and outputs (e.g. a variant-extraction module in eduomics).

## When NOT to use

- The step runs outside any workflow manager and a standalone script is sufficient
- You only need to run an existing published pipeline — use `nextflow run nf-core/eduomics` directly rather than implementing a new module
- The target pipeline does not follow the nf-core DSL2 one-container-per-process structure

## Inputs

- Nextflow samplesheet CSV (columns: id, type, chromosome, coverage, capture, reps, groups, simthreshold)
- Pipeline config parameters (e.g. --genome GATK.GRCh38, --outdir, container profile)
- Target capture-region BED / target chromosome definition
- Source database files (e.g. ClinVar records)

## Outputs

- Containerised Nextflow process definition implementing the analytical step
- Process-emitted artefacts following pipeline naming conventions (e.g. .vcf.gz/.tbi variant tracks)
- nf-test unit tests and passing CI/linting workflows

## How to apply

Write the step as one Nextflow process (or subworkflow) inside the eduomics pipeline, following the nf-core DSL2 convention of one container per process. Configure the container engine (Docker, Singularity, or Conda) and pin the reference genome, e.g. GATK.GRCh38, via pipeline parameters rather than hardcoding. Wire inputs as Nextflow channels (samplesheet rows carry id, type, chromosome, coverage, capture, reps, groups, simthreshold), and have the process emit outputs under the pipeline's naming convention (e.g. [simulation_id]_gnomad.vcf.gz with a .tbi). For module updates — as in SUBVAR recoding variant outputs to the reference format — propagate the change through the dependent subworkflow (SUBSET_REFERENCES_TO_TARGET was updated accordingly) and add nf-test unit tests, since the repository tracks CI and linting via GitHub Actions.

## Related tools

- **Nextflow** (Workflow framework in which the module is implemented as a DSL2 process (version >=25.04)) — https://github.com/lescai-teaching/eduomics
- **Docker** (Container engine used to execute the module process reproducibly) — https://github.com/lescai-teaching/eduomics
- **Singularity** (Alternative container engine for the module process) — https://github.com/lescai-teaching/eduomics
- **Conda** (Alternative non-container execution profile for the module process) — https://github.com/lescai-teaching/eduomics

## Examples

```
nextflow run nf-core/eduomics -profile docker --input dna_samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- The pipeline lints and runs CI successfully (GitHub Actions ci.yml and linting.yml badges pass)
- nf-test unit tests for the new module pass
- The process emits outputs in the expected format and naming convention, e.g. a loadable .vcf.gz/.tbi pair
- Variant outputs follow the reference genome format (GATK.GRCh38) rather than the source database format (ClinVar's), as verified after the SUBVAR update
- The module runs under all supported profiles (Docker, Singularity, Conda)

## Limitations

- Parameters must be provided via the CLI or -params-file; custom config files supplied with -c cannot set pipeline parameters
- Pipeline is chromosome-specific (typically chr22) and assumes human references, capture-based paired-end DNA simulation, so the module inherits that scope
- Processes should come from nf-core/modules where possible; hand-rolled containers increase maintenance burden

## Evidence

- [readme] DSL2 one-container-per-process: [evidence span withheld — anchor status: not_found]
- [readme] container engines: [evidence span withheld — anchor status: not_found]
- [discussion] SUBVAR reference-format update: "variant outputs now follow the reference format rather than ClinVar’s. The SUBSET_REFERENCES_TO_TARGET subworkflow was updated accordingly"
- [intro] ClinVar target-region extraction: "Extracts pathogenic variants from ClinVar database within your target regions."
