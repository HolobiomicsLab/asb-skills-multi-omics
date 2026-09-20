---
name: dna-read-simulation-with-variant-injection
description: Use when building educational genomic datasets where students must practice
  variant calling against data with known injected pathogenic variants, or when validating
  that simulated reads reproduce a specific ClinVar variant.
when_to_use_negative:
- The goal is RNA-seq differential expression simulation — use the RNA branch (Polyester/Salmon/DESeq2)
  instead of DNA variant simulation
- You need whole-genome scale data — the pipeline assumes chromosome-specific (typically
  chr22) capture-based exome/targeted sequencing to stay computationally tractable
- The biological ground truth of real patient data is the analysis target — this skill
  produces synthetic data, not clinical calls
edam_operation: http://edamontology.org/operation_3580
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0080
tools:
- name: Nextflow
  role: Workflow engine orchestrating the DNA simulation branch (version >=25.04.4)
  repo: https://github.com/lescai-teaching/eduomics
- name: SimuSCoP
  role: Generates paired-end FASTQ files with injected pathogenic variants using a
    BAM-derived sequencing profile (version 2.0.1)
  repo: https://github.com/lescai-teaching/eduomics
- name: Docker
  role: Containerized execution profile for reproducible pipeline runs
  repo: https://github.com/lescai-teaching/eduomics
- name: Singularity
  role: Alternative containerized execution profile
  repo: https://github.com/lescai-teaching/eduomics
- name: Conda
  role: Alternative dependency-resolution profile
  repo: https://github.com/lescai-teaching/eduomics
- name: FastQC
  role: Quality control of the simulated FASTQ files
  repo: https://github.com/lescai-teaching/eduomics
- name: MultiQC
  role: Aggregates QC reports for the simulation run
  repo: https://github.com/lescai-teaching/eduomics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:5d056d74fcbab7c468ab290b6195b4053d1c7bafbe5992e1eb1f20f5bfecc0d9
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
  edam_operation: http://edamontology.org/operation_3580
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# DNA read simulation with variant injection

## Summary

Simulates paired-end DNA FASTQ reads containing known pathogenic variants injected into a 'disease' sample using SimuSCoP 2.0.1 within the nf-core/eduomics Nextflow pipeline, producing teaching datasets with known ground truth for variant calling practice.

## When to use

Use when building educational genomic datasets where students must practice variant calling against data with known injected pathogenic variants, or when validating that simulated reads reproduce a specific ClinVar variant.

## When NOT to use

- The goal is RNA-seq differential expression simulation — use the RNA branch (Polyester/Salmon/DESeq2) instead of DNA variant simulation
- You need whole-genome scale data — the pipeline assumes chromosome-specific (typically chr22) capture-based exome/targeted sequencing to stay computationally tractable
- The biological ground truth of real patient data is the analysis target — this skill produces synthetic data, not clinical calls

## Inputs

- DNA samplesheet CSV (id, type, chromosome, coverage, capture BED URL, reps, groups, simthreshold)
- Human reference genome GATK.GRCh38 with annotation
- Capture regions BED file
- ClinVar database (pathogenic variants)
- gnomAD, Mills indels, 1000 Genomes, and dbSNP VCFs
- Existing BAM files for sequencing profile derivation

## Outputs

- Paired-end FASTQ files with injected pathogenic variants (disease sample) and matched normal sample reads
- Subset reference FASTA, BWA index, and capture BED files
- Region-subset VCFs: [simulation_id]_gnomad.vcf.gz, _mills.vcf.gz, _1000g.vcf.gz, _dbsnp.vcf.gz with .tbi indexes
- simulated_validated.vcf from the validation variant-calling step
- solution_[variant_name].txt answer key assembled into the dna_simulations output directory

## How to apply

Run nf-core/eduomics (Nextflow >=25.04.4) with a containerized profile (Docker, Singularity, or Conda), a DNA samplesheet (id, type, chromosome, coverage, capture, reps, groups, simthreshold), and GATK.GRCh38 as reference. First subset the reference genome and annotation to the target chromosome (typically chr22) and capture BED regions, building a BWA index; then extract pathogenic ClinVar variants within target regions and subset gnomAD, Mills indels, 1000 Genomes, and dbSNP VCFs to the same regions. Derive a sequencing profile from existing BAM files so simulated read characteristics are realistic, then run SimuSCoP 2.0.1 to generate paired-end FASTQ files with the selected pathogenic variant injected into the disease sample. Finally, perform variant calling on the simulated reads to confirm the injected variant is present in the disease sample and absent from the normal sample; only simulations passing this validation enter the final dna_simulations output alongside the solution_[variant_name].txt answer key.

## Related tools

- **Nextflow** (Workflow engine orchestrating the DNA simulation branch (version >=25.04.4)) — https://github.com/lescai-teaching/eduomics
- **SimuSCoP** (Generates paired-end FASTQ files with injected pathogenic variants using a BAM-derived sequencing profile (version 2.0.1)) — https://github.com/lescai-teaching/eduomics
- **Docker** (Containerized execution profile for reproducible pipeline runs) — https://github.com/lescai-teaching/eduomics
- **Singularity** (Alternative containerized execution profile) — https://github.com/lescai-teaching/eduomics
- **Conda** (Alternative dependency-resolution profile) — https://github.com/lescai-teaching/eduomics
- **FastQC** (Quality control of the simulated FASTQ files) — https://github.com/lescai-teaching/eduomics
- **MultiQC** (Aggregates QC reports for the simulation run) — https://github.com/lescai-teaching/eduomics

## Examples

```
nextflow run nf-core/eduomics -profile docker --input dna_samplesheet.csv --genome GATK.GRCh38
```

## Evaluation signals

- Variant calling on simulated reads (simulated_validated.vcf) must detect the injected pathogenic variant in the disease sample
- The same variant-calling check must show the pathogenic variant absent from the normal sample's reads
- Simulations failing validation are excluded from the final dna_simulations output — presence of a simulation in the final directory implies it passed
- All four region-subset VCFs ([simulation_id]_gnomad/mills/1000g/dbsnp .vcf.gz) exist with accompanying .tbi indexes
- A solution_[variant_name].txt answer key is emitted for each simulation, enabling students' calls to be checked against known truth

## Limitations

- Analysis is chromosome-specific (typically chr22) and assumes capture-based exome/targeted sequencing, so it does not represent whole-genome simulations
- SUBVAR module variant outputs were harmonised to reference format rather than ClinVar format, so downstream users should expect reference-style representations
- Only simulations passing the variant-calling validation are included in the final output; failure of the check silently excludes the simulation

## Evidence

- [intro] Read simulation with SimuSCoP and BAM-derived profile: "Uses SimuSCoP to generate paired-end FASTQ files containing the injected variants."
- [intro] Validation rationale for teaching suitability: "Checks that the simulated pathogenic variant is indeed present in the reads of the "disease" sample and absent from the "normal" sample"
- [intro] Filter step: only validated simulations retained: "Only simulations passing validation are included in the final output"
- [readme] Educational ground-truth philosophy: [evidence span withheld — anchor status: not_found]
- [readme] Assumption of capture-based, chromosome-specific sequencing: [evidence span withheld — anchor status: not_found]
- [intro] Reference genome recommendation: "**Reference genome configured** (we recommend using `GATK.GRCh38`)"
