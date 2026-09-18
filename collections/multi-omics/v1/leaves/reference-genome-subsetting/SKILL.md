---
name: reference-genome-subsetting
description: Use when a DNA simulation must target a specific chromosome and exome
  capture region, and subsetting workflow needs chromosome FASTA, padded BEDs, and
  region-restricted variant VCFs.
when_to_use_negative:
- Input is whole-genome analysis with no chromosome restriction — the skill is designed
  for single-chromosome, capture-based targeted sequencing (typically chr22) to reduce
  computational load
- The goal is selecting pathogenic ClinVar variants — that is a separate extraction
  step; this skill only subsets reference materials and general known-variant resources
- The target organism has no pre-indexed GATK.GRCh38-style reference or .dict/.fai
  support available
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Nextflow
  role: Implements the SUBSET_REFERENCES_TO_TARGET subworkflow that orchestrates the
    subsetting and indexing steps
  repo: https://github.com/lescai-teaching/eduomics
- name: samtools
  role: faidx-style FASTA indexing and chromosome-level FASTA subsetting
- name: bcftools
  role: Region-aware VCF subsetting (view -R) of gnomAD, Mills indels, 1000 Genomes,
    and dbSNP to padded target regions
- name: GATK
  role: Alternative for reference subsetting, SelectVariants region restriction, and
    .dict sequence dictionary creation
- name: Docker
  role: Container profile providing reproducible tool environments for the subworkflow
  repo: https://github.com/lescai-teaching/eduomics
merged_aliases:
- genomic-reference-subsetting-to-capture-regions
merged_alias_records:
- alias: genomic-reference-subsetting-to-capture-regions
  slug: genomic-reference-subsetting-to-capture-regions
  cosine_score: 0.8025
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:95b20ceec4cf5656cc6bc47c760a4bb2f5e86b61d74c4641c199dd4fd59f9c1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lescai-teaching/eduomics
  tools:
  - Conda
  - Docker
  - FastQC
  - GATK
  - MultiQC
  - Nextflow
  - SimuSCoP
  - Singularity
  - bcftools
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# reference-genome-subsetting

## Summary

Prepares per-simulation reference bundles in the eduomics pipeline by subsetting the GATK.GRCh38 genome, capture BED, and known-variant VCFs (gnomAD, Mills indels, 1000 Genomes, dbSNP) to a single chromosome and padded exome capture regions.

## When to use

Use when a DNA simulation must target a specific chromosome and exome capture region, and subsetting workflow needs chromosome FASTA, padded BEDs, and region-restricted variant VCFs.

## When NOT to use

- Input is whole-genome analysis with no chromosome restriction — the skill is designed for single-chromosome, capture-based targeted sequencing (typically chr22) to reduce computational load
- The goal is selecting pathogenic ClinVar variants — that is a separate extraction step; this skill only subsets reference materials and general known-variant resources
- The target organism has no pre-indexed GATK.GRCh38-style reference or .dict/.fai support available

## Inputs

- GATK.GRCh38 reference FASTA (recommended genome config in eduomics)
- Exome capture BED file of captured intervals
- Known-variant VCF resources: gnomAD, Mills indels, 1000 Genomes, dbSNP
- eduomics DNA samplesheet specifying id, chromosome (e.g. chr22), and capture BED URL

## Outputs

- [chromosome].fa with .fai and .dict companions
- Padded target BED files
- [simulation_id]_gnomad.vcf.gz, [simulation_id]_mills.vcf.gz, [simulation_id]_1000g.vcf.gz, [simulation_id]_dbsnp.vcf.gz (bgzipped)
- Matching .tbi tabix indexes for each subset VCF
- dna_simulations/[simulation_id]/references/ reference bundle (including BWA index for downstream simulation)

## How to apply

Load the GATK.GRCh38 reference FASTA and build the samtools-style .fai index and GATK .dict sequence dictionary. Subset the genome to the target chromosome (e.g. chr22 via samtools faidx or GATK Reference subsetting), emitting [chromosome].fa with its .fai and .dict companions. Pad the exome capture BED intervals to define the final target regions for the simulation. Subset the known-variant resources (gnomAD, Mills indels, 1000 Genomes, dbSNP) to the padded regions with a region-aware VCF tool such as bcftools view -R or GATK SelectVariants, writing bgzipped VCFs per simulation. Tabix-index each subset VCF so every [simulation_id]_* VCF is paired with its .tbi, and emit padded target BED files. Per the SUBSET_REFERENCES_TO_TARGET update, variant outputs follow the reference resource format rather than ClinVar's format.

## Related tools

- **Nextflow** (Implements the SUBSET_REFERENCES_TO_TARGET subworkflow that orchestrates the subsetting and indexing steps) — https://github.com/lescai-teaching/eduomics
- **samtools** (faidx-style FASTA indexing and chromosome-level FASTA subsetting)
- **bcftools** (Region-aware VCF subsetting (view -R) of gnomAD, Mills indels, 1000 Genomes, and dbSNP to padded target regions)
- **GATK** (Alternative for reference subsetting, SelectVariants region restriction, and .dict sequence dictionary creation)
- **Docker** (Container profile providing reproducible tool environments for the subworkflow) — https://github.com/lescai-teaching/eduomics

## Examples

```
nextflow run eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- file_exists / file_format_is checks pass for *_gnomad.vcf.gz, *_mills.vcf.gz, *_1000g.vcf.gz, *_dbsnp.vcf.gz and their .tbi indexes
- Each [chromosome].fa is accompanied by its .fai and .dict companions
- Every subset VCF record falls within the padded target BED intervals
- Variant outputs conform to the reference resource format, not ClinVar's format (per the SUBSET_REFERENCES_TO_TARGET update)
- The complete reference bundle (FASTA set, padded BEDs, four subset/indexed VCFs) exits the subworkflow per simulation

## Limitations

- Designed for single-chromosome, capture-based (exome/targeted) human sequencing; not intended for whole-genome or non-human references
- Chromosome choice is typically chr22 to keep computational requirements manageable while maintaining biological realism
- Subsetting depends on correctly padded capture intervals — unpadded BEDs would miss variants at interval edges in downstream variant calling

## Evidence

- [intro] Reference preparation subsets the genome and annotation: "**Reference Preparation**: Subsets the reference genome and annotation to your target chromosome and capture regions."
- [discussion] SUBSET_REFERENCES_TO_TARGET follows reference format: "variant outputs now follow the reference format rather than ClinVar’s. The SUBSET_REFERENCES_TO_TARGET subworkflow was updated accordingly"
- [intro] Subset VCF outputs with tabix indexes: "**`[simulation_id]_gnomad.vcf.gz`** and **`[simulation_id]_gnomad.vcf.gz.tbi`**: gnomAD variants"
- [readme] README mermaid shows the subsetting stage in the DNA branch: [evidence span withheld — anchor status: not_found]
