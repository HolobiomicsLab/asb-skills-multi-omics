---
name: fasta-indexing-and-sequence-dictionary-generation
description: Use when a GATK-compatible reference genome (e.g. GATK.GRCh38) must be
  made addressable for subsetting (samtools faidx / GATK Reference subsetting) or
  region-aware VCF tools like bcftools view -R before per-simulation reference bundles
  are built.
when_to_use_negative:
- Inputs are already indexed FASTQ or VCF files — this skill applies only to reference
  genome FASTAs needing .fai/.dict companions
- RNA-seq simulations, which subset transcriptome references rather than genomic FASTA/capture-region
  references
- You only need ClinVar-format variant extraction without building the per-simulation
  reference bundle (outputs deliberately keep the reference resource format, not ClinVar's)
tools:
- name: samtools
  role: faidx indexing of the reference FASTA and subsetting to the target chromosome,
    emitting [chromosome].fa with .fai
- name: GATK
  role: sequence dictionary (.dict) creation and Reference subsetting; GATK.GRCh38
    is the recommended reference genome
- name: Nextflow
  role: orchestrates the SUBSET_REFERENCES_TO_TARGET subworkflow that channels indexed
    FASTA, BEDs, and subset VCFs into the reference bundle
  repo: https://github.com/lescai-teaching/eduomics
- name: Docker
  role: container system used to run the pipeline steps reproducibly
  repo: https://github.com/lescai-teaching/eduomics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:6139530baa523c79c8b55fb9cf0401944ae5ffcdbb54323e7211e42ebda16d89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lescai-teaching/eduomics
  tools:
  - Docker
  - GATK
  - Nextflow
  - samtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# FASTA indexing and sequence dictionary generation

## Summary

Prepares a reference genome FASTA for downstream subsetting and GATK-based tooling by generating the samtools .fai index and the GATK .dict sequence dictionary, as the first step of eduomics' SUBSET_REFERENCES_TO_TARGET subworkflow for chromosome/capture-region reference preparation.

## When to use

Use when a GATK-compatible reference genome (e.g. GATK.GRCh38) must be made addressable for subsetting (samtools faidx / GATK Reference subsetting) or region-aware VCF tools like bcftools view -R before per-simulation reference bundles are built.

## When NOT to use

- Inputs are already indexed FASTQ or VCF files — this skill applies only to reference genome FASTAs needing .fai/.dict companions
- RNA-seq simulations, which subset transcriptome references rather than genomic FASTA/capture-region references
- You only need ClinVar-format variant extraction without building the per-simulation reference bundle (outputs deliberately keep the reference resource format, not ClinVar's)

## Inputs

- Reference genome FASTA (GATK.GRCh38 recommended) configured via --genome
- Target chromosome identifier from the samplesheet (e.g. chr22)

## Outputs

- Reference .fai index (samtools-style)
- Reference .dict sequence dictionary (GATK)
- Subset [chromosome].fa with its .fai and .dict companions
- Per-simulation reference bundle under dna_simulations/[simulation_id]/references/

## How to apply

Load the reference FASTA configured for eduomics — the pipeline recommends `GATK.GRCh38` — and create two companion index files: the samtools-style .fai index (enabling random-access faidx extraction of the target chromosome, e.g. emitting [chromosome].fa) and the GATK .dict sequence dictionary (required by GATK tools and region-aware subsetting). These indexed FASTAs are then subset to the target chromosome (e.g. chr22 per the samplesheet) with samtools faidx or GATK Reference subsetting, each [chromosome].fa carrying its own .fai and .dict companions. The indexed, subset FASTA is later channelled together with padded capture-region BEDs and tabix-indexed subset VCFs (gnomAD, Mills, 1000G, dbSNP) into the per-simulation reference bundle under dna_simulations/[simulation_id]/references/. Note that variant outputs follow the reference resource format rather than ClinVar's, per the SUBSET_REFERENCES_TO_TARGET update.

## Related tools

- **samtools** (faidx indexing of the reference FASTA and subsetting to the target chromosome, emitting [chromosome].fa with .fai)
- **GATK** (sequence dictionary (.dict) creation and Reference subsetting; GATK.GRCh38 is the recommended reference genome)
- **Nextflow** (orchestrates the SUBSET_REFERENCES_TO_TARGET subworkflow that channels indexed FASTA, BEDs, and subset VCFs into the reference bundle) — https://github.com/lescai-teaching/eduomics
- **Docker** (container system used to run the pipeline steps reproducibly) — https://github.com/lescai-teaching/eduomics

## Examples

```
samtools faidx GRCh38.chr22.fa && gatk CreateSequenceDictionary -R GRCh38.chr22.fa
```

## Evaluation signals

- The .fai and .dict files exist alongside every reference/[chromosome].fa emitted by the subworkflow (file_exists check)
- Each known-variant VCF in the reference bundle has a matching .tbi index: *_gnomad.vcf.gz, *_mills.vcf.gz, *_1000g.vcf.gz, *_dbsnp.vcf.gz plus .tbi (file_exists / file_format_is)
- samtools faidx can extract sequences from the subset [chromosome].fa using the .fai index without errors
- GATK tools accept the subset FASTA, implying a valid, contig-consistent .dict sequence dictionary
- The subworkflow's outputs (FASTA trio, padded BEDs, four indexed VCFs) are all channelled out of SUBSET_REFERENCES_TO_TARGET into the per-simulation bundle

## Limitations

- Indexing must be redone per reference copy: subsetting the genome to the target chromosome requires regenerating .fai/.dict companions for the [chromosome].fa output
- SUBVAR variant outputs now follow the reference resource format rather than ClinVar's — older ClinVar-format expectations will fail validation
- Designed for single-chromosome (typically chr22), capture-based DNA simulations; whole-genome or RNA branch references follow different subsetting logic

## Evidence

- [other] Reference preparation steps for eduomics DNA simulations: [evidence span withheld — anchor status: not_found]
- [other] Subset FASTA output companions: [evidence span withheld — anchor status: not_found]
- [readme] README reference preparation feature: "**Reference Preparation**: Subsets the reference genome and annotation to your target chromosome and capture regions."
- [discussion] SUBVAR format update tied to SUBSET_REFERENCES_TO_TARGET: "variant outputs now follow the reference format rather than ClinVar’s. The SUBSET_REFERENCES_TO_TARGET subworkflow was updated accordingly"
