---
name: pathogenic-variant-detection-verification
description: Use when simulated DNA reads carry injected ClinVar pathogenic variants
  and you must verify, via variant calling against a reference bundle, that the ground-truth
  variant is recovered before releasing the dataset for teaching.
when_to_use_negative:
- Input is real patient sequencing data rather than simulated reads with a known injected
  variant ground truth — this step is specific to validating eduomics simulations
- The goal is RNA-seq differential expression validation (eduomics handles that via
  Salmon/DESeq2, not DNA variant calling)
- You need discovery of novel variants; this step only verifies recovery of a known
  injected pathogenic variant
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Nextflow
  role: Workflow engine that launches the DNAVALIDATION module (version >=25.04.4)
  repo: https://www.nextflow.io/
- name: Docker
  role: Container engine profile for reproducible execution of the validation module
- name: Singularity
  role: Alternative container engine profile for the validation run
- name: Conda
  role: Alternative dependency environment profile for the validation run
- name: FastQC
  role: Quality control of the simulated disease/normal FASTQ files before variant
    calling
- name: MultiQC
  role: Aggregates FastQC reports into a summary prior to calling
- name: SimuSCoP
  role: Upstream tool that generated the paired-end FASTQ files containing the injected
    variants being validated
  repo: https://github.com/lescai-teaching/eduomics
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:31d852fe076b4bc1a003c96771642e00e9f8c19feea07ab357cdf405a476b748
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Conda
  - Docker
  - FastQC
  - MultiQC
  - Nextflow
  - SimuSCoP
  - Singularity
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# pathogenic-variant-detection-verification

## Summary

A validation step from the eduomics DNA simulation branch that performs variant calling on simulated paired-end FASTQ reads (disease and normal samples) to confirm the injected pathogenic variant is detectable in the disease sample and absent from the normal sample, emitting simulated_validated.vcf.gz.

## When to use

Use when simulated DNA reads carry injected ClinVar pathogenic variants and you must verify, via variant calling against a reference bundle, that the ground-truth variant is recovered before releasing the dataset for teaching.

## When NOT to use

- Input is real patient sequencing data rather than simulated reads with a known injected variant ground truth — this step is specific to validating eduomics simulations
- The goal is RNA-seq differential expression validation (eduomics handles that via Salmon/DESeq2, not DNA variant calling)
- You need discovery of novel variants; this step only verifies recovery of a known injected pathogenic variant

## Inputs

- Simulated paired-end FASTQ files for the disease sample (with injected variant, from SimuSCoP)
- Simulated paired-end FASTQ files for the normal sample
- Subset reference bundle derived from GATK.GRCh38 (genome and annotation subset to target chromosome/capture regions)
- Injected pathogenic variant truth set (extracted from ClinVar within target regions)

## Outputs

- simulated_validated.vcf.gz (validated variant VCF in the variant-specific output directory)
- FastQC and MultiQC quality reports on the simulated reads
- Validation verdict per simulation (pass/fail gating final output)

## How to apply

Launch the nf-core/eduomics DNAVALIDATION module with Nextflow (>=25.04.4) under a container engine profile (Docker, Singularity, or Conda). Run FastQC on the simulated disease and normal paired-end FASTQ files and aggregate with MultiQC before calling. Align the simulated reads and perform variant calling against the subset reference bundle derived from GATK.GRCh38 to produce a VCF for both samples. Inspect the called variants to confirm the injected pathogenic variant is present in the disease sample and absent from the normal sample, then emit simulated_validated.vcf.gz in the variant-specific output directory and record the validation verdict. Only simulations passing validation are included in the final pipeline output, so a failed check means the simulation is excluded downstream.

## Related tools

- **Nextflow** (Workflow engine that launches the DNAVALIDATION module (version >=25.04.4)) — https://www.nextflow.io/
- **Docker** (Container engine profile for reproducible execution of the validation module)
- **Singularity** (Alternative container engine profile for the validation run)
- **Conda** (Alternative dependency environment profile for the validation run)
- **FastQC** (Quality control of the simulated disease/normal FASTQ files before variant calling)
- **MultiQC** (Aggregates FastQC reports into a summary prior to calling)
- **SimuSCoP** (Upstream tool that generated the paired-end FASTQ files containing the injected variants being validated) — https://github.com/lescai-teaching/eduomics

## Examples

```
nextflow run nf-core/eduomics -profile docker --input dna_samplesheet.csv --genome GATK.GRCh38
```

## Evaluation signals

- The injected pathogenic variant is called in the disease-sample VCF
- The injected variant is absent from the normal-sample VCF
- simulated_validated.vcf.gz is emitted in the variant-specific output directory
- FastQC/MultiQC reports show simulated reads meet quality standards before calling
- The simulation's validation verdict is 'pass'; failures are excluded from the final output

## Limitations

- Only simulations passing validation are included in the final pipeline output, so a failed validation removes the dataset entirely
- Analysis is chromosome-specific (typically chr22) and assumes capture-based (exome/targeted) paired-end sequencing against a human reference
- Validates only the injected variant ground truth; it does not assess sensitivity to other variant classes

## Evidence

- [intro] Checks that the simulated pathogenic variant is indeed present in the reads of the "disease" sample and absent from the "normal" sample: "Checks that the simulated pathogenic variant is indeed present in the reads of the "disease" sample and absent from the "normal" sample"
- [intro] Validation: performs variant calling to verify injected variants are detectable: "**Validation**: Performs variant calling to verify that injected variants can be detected."
- [results] Only simulations passing validation are included in the final output: "Only simulations passing validation are included in the final output"
- [readme] Students can practice variant calling knowing exactly which variants should be detected: [evidence span withheld — anchor status: not_found]
