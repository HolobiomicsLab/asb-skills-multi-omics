---
name: vcf-region-subsetting-and-indexing
description: Use when preparing reference materials for a chromosome/capture-region-targeted
  DNA simulation that needs known variant databases (gnomAD, Mills, 1000G, dbSNP)
  restricted to padded target intervals and paired with .tbi indexes.
when_to_use_negative:
- You are preparing the RNA-seq simulation branch, which subsets transcriptome references
  rather than variant VCFs
- You need ClinVar-format variant outputs — the SUBSET_REFERENCES_TO_TARGET subworkflow
  was updated so variant outputs follow the reference resource format instead of ClinVar's
- The goal is whole-genome variant annotation with no chromosome/capture-region restriction,
  where subsetting to padded target regions is unnecessary
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Nextflow
  role: Workflow engine running the SUBSET_REFERENCES_TO_TARGET subworkflow within
    eduomics
  repo: https://github.com/lescai-teaching/eduomics
- name: Docker
  role: Container system providing reproducible environments for the subsetting tools
  repo: https://www.docker.com/
- name: bcftools
  role: Region-aware VCF subsetting of known-variant resources to padded target regions
    (bcftools view -R)
- name: GATK
  role: Alternative subsetting via GATK SelectVariants / Reference subsetting; source
    of the recommended GATK.GRCh38 reference bundle
- name: samtools
  role: samtools faidx chromosome-level FASTA subsetting and .fai indexing
- name: Tabix
  role: Indexes each subset bgzipped VCF as .tbi so VCFs are region-queryable
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:867a8fdf54338149551e7632ce5e07d1ba0a7d1e2c276b591457d40ae41ee09e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lescai-teaching/eduomics
  tools:
  - Docker
  - GATK
  - Nextflow
  - Tabix
  - bcftools
  - samtools
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# VCF region subsetting and indexing

## Summary

Subsets known-variant VCF resources (gnomAD, Mills indels, 1000 Genomes, dbSNP) to padded capture-region targets and bgzip/tabix-indexes them so each eduomics DNA simulation receives a per-simulation reference bundle with region-restricted, queryable variant files.

## When to use

Use when preparing reference materials for a chromosome/capture-region-targeted DNA simulation that needs known variant databases (gnomAD, Mills, 1000G, dbSNP) restricted to padded target intervals and paired with .tbi indexes.

## When NOT to use

- You are preparing the RNA-seq simulation branch, which subsets transcriptome references rather than variant VCFs
- You need ClinVar-format variant outputs — the SUBSET_REFERENCES_TO_TARGET subworkflow was updated so variant outputs follow the reference resource format instead of ClinVar's
- The goal is whole-genome variant annotation with no chromosome/capture-region restriction, where subsetting to padded target regions is unnecessary

## Inputs

- GATK.GRCh38 reference FASTA (with .fai and .dict companions)
- Exome capture BED file of captured intervals (to be padded into target regions)
- Known-variant VCF resources: gnomAD, Mills indels, 1000 Genomes, dbSNP
- Per-simulation parameters: target chromosome (e.g. chr22) and simulation_id

## Outputs

- [chromosome].fa with .fai and .dict sequence dictionary files
- Padded target BED files per simulation
- [simulation_id]_gnomad.vcf.gz and .tbi
- [simulation_id]_mills.vcf.gz and .tbi
- [simulation_id]_1000g.vcf.gz and .tbi
- [simulation_id]_dbsnp.vcf.gz and .tbi
- Per-simulation reference bundle under dna_simulations/[simulation_id]/references/

## How to apply

Load the GATK.GRCh38 reference FASTA (eduomics recommends `GATK.GRCh38`) and build the samtools-style .fai index plus the GATK .dict sequence dictionary. Subset the genome to the target chromosome (e.g. samtools faidx or GATK Reference subsetting), emitting [chromosome].fa with its .fai and .dict companions. Take the exome capture BED, pad the captured intervals to define the final target regions, then subset each known-variant resource to the padded regions with a region-aware VCF tool (e.g. bcftools view -R or GATK SelectVariants), writing bgzipped VCFs per simulation. Tabix-index every subset VCF so each `[simulation_id]_*` VCF is paired with its .tbi, and channel the [chromosome].fa/.fai/.dict, padded BEDs, and the four subset/indexed VCFs out as the per-simulation reference bundle. Per the SUBSET_REFERENCES_TO_TARGET update, keep the reference resource format for variant outputs rather than ClinVar's format.

## Related tools

- **Nextflow** (Workflow engine running the SUBSET_REFERENCES_TO_TARGET subworkflow within eduomics) — https://github.com/lescai-teaching/eduomics
- **Docker** (Container system providing reproducible environments for the subsetting tools) — https://www.docker.com/
- **bcftools** (Region-aware VCF subsetting of known-variant resources to padded target regions (bcftools view -R))
- **GATK** (Alternative subsetting via GATK SelectVariants / Reference subsetting; source of the recommended GATK.GRCh38 reference bundle)
- **samtools** (samtools faidx chromosome-level FASTA subsetting and .fai indexing)
- **Tabix** (Indexes each subset bgzipped VCF as .tbi so VCFs are region-queryable)

## Examples

```
bcftools view -R targets.padded.bed -Oz -o ${simulation_id}_gnomad.vcf.gz gnomad.vcf.gz && tabix -p vcf ${simulation_id}_gnomad.vcf.gz
```

## Evaluation signals

- Each of *_gnomad.vcf.gz, *_mills.vcf.gz, *_1000g.vcf.gz, *_dbsnp.vcf.gz has a matching .tbi index (file_exists / file_format_is checks)
- Subworkflow outputs exist for every [simulation_id]: [chromosome].fa with .fai/.dict companions, padded target BEDs, and the four subset VCFs
- All subset VCF records fall within the padded target intervals derived from the capture BED
- Variant outputs use the reference resource format rather than ClinVar's format
- Reference bundle is emitted under dna_simulations/[simulation_id]/references/

## Limitations

- Pipeline targets single chromosomes (typically chr22) with capture-based sequencing, so subsetting is designed for chromosome-specific, capture-region analysis rather than genome-wide inputs
- Reference genome must be a standard human reference such as GATK.GRCh38; other assemblies are not covered by the eduomics defaults
- Variant resources must be pre-existing known-variant databases (gnomAD, Mills, 1000 Genomes, dbSNP); the subworkflow subsets rather than creates variant records

## Evidence

- [intro] Reference Preparation workflow step: "Reference Preparation: Subsets the reference genome and annotation to your target chromosome and capture regions."
- [discussion] SUBSET_REFERENCES_TO_TARGET format update: "variant outputs now follow the reference format rather than ClinVar’s. The SUBSET_REFERENCES_TO_TARGET subworkflow was updated accordingly"
- [readme] gnomAD output pair naming: "**`[simulation_id]_gnomad.vcf.gz`** and **`[simulation_id]_gnomad.vcf.gz.tbi`**: gnomAD variants"
- [other] Per-simulation reference bundle: [evidence span withheld — anchor status: not_found]
