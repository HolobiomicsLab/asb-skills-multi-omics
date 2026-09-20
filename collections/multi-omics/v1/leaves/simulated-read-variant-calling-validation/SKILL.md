---
name: simulated-read-variant-calling-validation
description: Use when simulated sequencing reads have been generated with an injected
  pathogenic variant and the dataset must be certified for educational variant-calling
  exercises before release to students.
when_to_use_negative:
- The reads are real patient/study sequencing data rather than eduomics simulations
  with a known injected variant — no ground truth exists to validate against
- The goal is RNA-seq simulation validation (differential expression patterns via
  DESeq2/Salmon), which uses a separate validation branch
- 'You need clinical-grade variant calling: this check only certifies detectability
  of a known injected variant for teaching, not diagnostic accuracy'
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3168
tools:
- name: Nextflow
  role: Workflow engine orchestrating the validation step (nf-core/eduomics, version
    >=25.04.4)
  repo: https://github.com/lescai-teaching/eduomics
- name: SimuSCoP
  role: Generates the paired-end FASTQ reads with the injected pathogenic variant
    that are subsequently validated by variant calling
  repo: https://github.com/lescai-teaching/eduomics
- name: Docker
  role: Containerized execution profile for reproducible pipeline runs
  repo: https://github.com/lescai-teaching/eduomics
- name: FastQC
  role: Quality control of FASTQ files supporting simulated-data quality standards
  repo: https://github.com/lescai-teaching/eduomics
- name: MultiQC
  role: Aggregates QC reports (with FastQC) for the simulated datasets
  repo: https://github.com/lescai-teaching/eduomics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:1c295a0f68e7c809f6adfd42955944754cbee82e11e1c2372238404d02b0a682
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
  - SimuSCoP
  - Singularity
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# simulated-read-variant-calling-validation

## Summary

A quality-gate step in the eduomics DNA simulation workflow that performs variant calling on SimuSCoP-generated paired-end FASTQ reads to confirm the injected pathogenic ClinVar variant is detectable in the disease sample and absent from the normal sample, ensuring the simulated dataset is suitable for teaching variant calling workflows.

## When to use

Use when simulated sequencing reads have been generated with an injected pathogenic variant and the dataset must be certified for educational variant-calling exercises before release to students.

## When NOT to use

- The reads are real patient/study sequencing data rather than eduomics simulations with a known injected variant — no ground truth exists to validate against
- The goal is RNA-seq simulation validation (differential expression patterns via DESeq2/Salmon), which uses a separate validation branch
- You need clinical-grade variant calling: this check only certifies detectability of a known injected variant for teaching, not diagnostic accuracy

## Inputs

- SimuSCoP-generated paired-end FASTQ files for disease and normal samples (with injected pathogenic variant)
- Subset reference genome FASTA with BWA index
- Capture region BED files
- Subset variant VCFs and .tbi indexes: [simulation_id]_gnomad.vcf.gz, [simulation_id]_mills.vcf.gz, [simulation_id]_1000g.vcf.gz, [simulation_id]_dbsnp.vcf.gz
- Simulated variant identity (pathogenic ClinVar variant within target regions, e.g. chr22)

## Outputs

- simulated_validated.vcf confirming the injected pathogenic variant is present in the disease sample and absent from the normal sample
- solution_[variant_name].txt answer key for the educational scenario
- dna_simulations output directory containing only simulations that passed validation

## How to apply

After SimuSCoP 2.0.1 generates paired-end FASTQ files with the selected pathogenic variant injected into the disease sample, run variant calling on the simulated reads against the subset reference (GATK.GRCh38-derived FASTA with BWA index and capture BED files). Check that the injected pathogenic variant is called in the disease sample's reads and is absent from the normal sample's reads, emitting a simulated_validated.vcf. Treat the variant-calling check as an acceptance filter: only simulations passing validation are included in the final dna_simulations output directory; failing simulations are excluded. Pair the validated VCF with the solution_[variant_name].txt answer key so instructors can verify student variant-calling results against known ground truth.

## Related tools

- **Nextflow** (Workflow engine orchestrating the validation step (nf-core/eduomics, version >=25.04.4)) — https://github.com/lescai-teaching/eduomics
- **SimuSCoP** (Generates the paired-end FASTQ reads with the injected pathogenic variant that are subsequently validated by variant calling) — https://github.com/lescai-teaching/eduomics
- **Docker** (Containerized execution profile for reproducible pipeline runs) — https://github.com/lescai-teaching/eduomics
- **FastQC** (Quality control of FASTQ files supporting simulated-data quality standards) — https://github.com/lescai-teaching/eduomics
- **MultiQC** (Aggregates QC reports (with FastQC) for the simulated datasets) — https://github.com/lescai-teaching/eduomics

## Examples

```
nextflow run eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- The injected pathogenic ClinVar variant is detected by variant calling in the disease sample's reads
- The same variant is absent from the normal sample's reads
- A simulated_validated.vcf is emitted containing the validated variant call
- Only simulations passing validation appear in the final dna_simulations output directory; failed simulations are excluded
- The simulation passes FastQC/MultiQC quality checks before educational release

## Limitations

- Simulations that fail the variant-detection validation are excluded from the final output, so coverage or simulation parameters must be sufficient for the injected variant to be callable
- The pipeline is chromosome-specific (typically chr22) and assumes capture-based (exome/targeted) paired-end sequencing on the human GATK.GRCh38 reference, which bounds where variants can be validated

## Evidence

- [intro] Validation of simulated disease reads: "Checks that the simulated pathogenic variant is indeed present in the reads of the "disease" sample and absent from the "normal" sample"
- [intro] Validation acceptance filter: "Only simulations passing validation are included in the final output"
- [intro] Variant calling purpose: "Performs variant calling to verify that injected variants can be detected."
- [readme] Validation built into pipeline: [evidence span withheld — anchor status: not_found]
- [readme] Pipeline DNA branch architecture: [evidence span withheld — anchor status: not_found]
