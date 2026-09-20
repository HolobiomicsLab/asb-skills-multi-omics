---
name: dna-variant-calling-from-simulated-reads
description: Use when you have simulated paired-end DNA FASTQ files with a known injected
  ClinVar pathogenic variant (disease vs normal) and must verify the variant is recoverable
  by variant calling before using the data for teaching variant-calling workflows.
when_to_use_negative:
- Input is real patient sequencing data rather than eduomics-simulated reads with
  a known ground-truth injected variant — there is no injected variant to validate
  against
- The goal is RNA differential expression validation (eduomics RNA branch) rather
  than DNA variant detection
- The reference genome or target region differs from the pipeline's human standard
  reference (e.g. non-human data or no GATK.GRCh38-derived subset bundle available)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Nextflow
  role: Workflow engine used to run the DNAVALIDATION module (version >=25.04.4)
  repo: https://github.com/lescai-teaching/eduomics
- name: nf-core/eduomics
  role: Pipeline containing the DNAVALIDATION validation step executed via nextflow
    run
  repo: https://github.com/lescai-teaching/eduomics
- name: FastQC
  role: Quality control of the simulated disease and normal paired-end FASTQ files
    before variant calling
- name: MultiQC
  role: Aggregates the FastQC quality reports prior to variant calling
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:9a481d091047f2a40acfd2fd159b5e0e0c336289bdf1a3ad055e22f8dbc34950
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lescai-teaching/eduomics
  tools:
  - Conda
  - Docker
  - FastQC
  - MultiQC
  - Nextflow
  - Singularity
  - nf-core/eduomics
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# dna-variant-calling-from-simulated-reads

## Summary

Validation step of the eduomics DNA simulation branch: variant calling is run on SimuSCoP-generated paired-end FASTQ reads (disease and normal samples) to confirm the injected pathogenic variant is detectable, producing simulated_validated.vcf.gz. Only simulations passing this validation are included in the final pipeline output.

## When to use

Use when you have simulated paired-end DNA FASTQ files with a known injected ClinVar pathogenic variant (disease vs normal) and must verify the variant is recoverable by variant calling before using the data for teaching variant-calling workflows.

## When NOT to use

- Input is real patient sequencing data rather than eduomics-simulated reads with a known ground-truth injected variant — there is no injected variant to validate against
- The goal is RNA differential expression validation (eduomics RNA branch) rather than DNA variant detection
- The reference genome or target region differs from the pipeline's human standard reference (e.g. non-human data or no GATK.GRCh38-derived subset bundle available)

## Inputs

- Simulated paired-end FASTQ files for a disease sample and a normal sample (produced by SimuSCoP with injected variants)
- Subset reference bundle derived from GATK.GRCh38 (restricted to target chromosome/capture regions)
- Injected pathogenic variant calls extracted from ClinVar within target regions

## Outputs

- simulated_validated.vcf.gz (validated variant VCF in the variant-specific output directory)
- Validation verdict stating whether the pathogenic variant is present in the disease sample and absent from the normal sample
- FastQC and MultiQC quality-control reports of the simulated FASTQ files

## How to apply

Launch the nf-core/eduomics DNAVALIDATION module under Nextflow (>=25.04.4) with a container profile (Docker, Singularity, or Conda). Run FastQC on the simulated disease and normal paired-end FASTQ files and aggregate the reports with MultiQC before calling. Align the reads and perform variant calling against the subset GATK.GRCh38-derived reference bundle to produce a VCF for both samples. Inspect the called variants to confirm the injected pathogenic variant is present in the disease sample and absent from the normal sample, then emit simulated_validated.vcf.gz in the variant-specific output directory and record the validation verdict. The rationale is that eduomics datasets must carry known, verifiable ground truth; only simulations passing validation flow through to the final output.

## Related tools

- **Nextflow** (Workflow engine used to run the DNAVALIDATION module (version >=25.04.4)) — https://github.com/lescai-teaching/eduomics
- **nf-core/eduomics** (Pipeline containing the DNAVALIDATION validation step executed via nextflow run) — https://github.com/lescai-teaching/eduomics
- **FastQC** (Quality control of the simulated disease and normal paired-end FASTQ files before variant calling)
- **MultiQC** (Aggregates the FastQC quality reports prior to variant calling)

## Examples

```
nextflow run nf-core/eduomics -profile docker --input dna_samplesheet.csv --genome GATK.GRCh38
```

## Evaluation signals

- The injected pathogenic variant is called as present in the disease-sample VCF and absent from the normal-sample VCF
- simulated_validated.vcf.gz is emitted in the variant-specific output directory
- FastQC/MultiQC reports confirm the simulated paired-end FASTQ files pass quality checks before calling
- Only simulations passing the validation verdict appear in the final pipeline output
- Called variants are consistent with the GATK.GRCh38-derived subset reference bundle used for calling

## Limitations

- Pipeline is designed for human data using standard reference genomes (GATK.GRCh38 recommended) and single-chromosome (typically chr22), capture-based paired-end simulations
- Only simulations passing validation are included in the final output, so failing simulations yield no downstream educational package
- Validation depends on the subset reference bundle; variant outputs follow the reference format rather than ClinVar's format (SUBVAR harmonisation)

## Evidence

- [intro] Validation performs variant calling on simulated reads: "**Validation**: Performs variant calling to verify that injected variants can be detected."
- [intro] Validation confirms variant presence per sample: "Checks that the simulated pathogenic variant is indeed present in the reads of the "disease" sample and absent from the "normal" sample"
- [intro] Only validated simulations reach final output: "- Only simulations passing validation are included in the final output"
- [readme] Pipeline runs via Nextflow with container profiles: [evidence span withheld — anchor status: not_found]
