---
name: nextflow-subworkflow-reconstruction
description: Use when a Nextflow DNA simulation pipeline must subset reference genome
  and known-variant resources to a target chromosome plus padded exome capture regions
  before read simulation.
when_to_use_negative:
- Input is an RNA simulation (type=rna) — the RNA branch subsets transcriptome references
  instead of genome/variant resources
- You need ClinVar-format variant outputs — the SUBSET_REFERENCES_TO_TARGET update
  intentionally emits variants in the reference resource format rather than ClinVar's
- Analysis spans multiple chromosomes or whole genome — the pipeline is designed for
  single-chromosome (typically chr22), capture-based simulations
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Nextflow
  role: Workflow engine implementing the SUBSET_REFERENCES_TO_TARGET subworkflow in
    DSL2
  repo: https://www.nextflow.io/
- name: Docker
  role: Container system providing reproducible environments for the subsetting processes
  repo: https://www.docker.com/
- name: samtools
  role: faidx indexing and chromosome-level FASTA subsetting
- name: bcftools
  role: Region-aware subsetting of known-variant VCFs with `bcftools view -R`
- name: GATK
  role: Sequence dictionary creation and SelectVariants-based VCF subsetting to padded
    target regions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:1245afc07547dee1b4a3f1c5c8f0add1d395618dfee023a638f1dc8619eaa633
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - GATK
  - Nextflow
  - bcftools
  - samtools
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# nextflow-subworkflow-reconstruction

## Summary

Reconstructs the SUBSET_REFERENCES_TO_TARGET subworkflow of the eduomics Nextflow pipeline, which prepares per-simulation reference bundles (chromosome FASTA, padded capture BEDs, and region-subset known-variant VCFs) for each DNA simulation targeting a specific chromosome and capture region.

## When to use

Use when a Nextflow DNA simulation pipeline must subset reference genome and known-variant resources to a target chromosome plus padded exome capture regions before read simulation.

## When NOT to use

- Input is an RNA simulation (type=rna) — the RNA branch subsets transcriptome references instead of genome/variant resources
- You need ClinVar-format variant outputs — the SUBSET_REFERENCES_TO_TARGET update intentionally emits variants in the reference resource format rather than ClinVar's
- Analysis spans multiple chromosomes or whole genome — the pipeline is designed for single-chromosome (typically chr22), capture-based simulations

## Inputs

- GATK.GRCh38 reference FASTA (plus .fai and .dict indices)
- Exome capture BED file of captured intervals (e.g. capture.bed from the samplesheet)
- Known-variant VCF resources: gnomAD, Mills indels, 1000 Genomes, dbSNP
- eduomics samplesheet row with id, type=dna, chromosome (e.g. chr22), and capture region

## Outputs

- Subset chromosome FASTA ([chromosome].fa) with .fai and .dict companions
- Padded target-region BED files
- Per-simulation bgzipped VCFs: [simulation_id]_gnomad.vcf.gz, [simulation_id]_mills.vcf.gz, [simulation_id]_1000g.vcf.gz, [simulation_id]_dbsnp.vcf.gz
- Tabix indexes (.tbi) for each subset VCF, emitted under dna_simulations/[simulation_id]/references/

## How to apply

Load the GATK.GRCh38 reference FASTA (eduomics recommends `--genome GATK.GRCh38`) and build the samtools-style .fai index plus the GATK .dict sequence dictionary. Subset the genome to the single target chromosome (typically chr22, e.g. with samtools faidx or GATK reference subsetting), emitting [chromosome].fa with its .fai and .dict companions. Pad the exome capture BED intervals to define the final target regions, then subset the four known-variant resources (gnomAD, Mills indels, 1000 Genomes, dbSNP) to those padded regions with a region-aware VCF tool such as `bcftools view -R` or GATK SelectVariants, writing bgzipped VCFs per simulation. Tabix-index each subset VCF so every `[simulation_id]_*` VCF is paired with its .tbi, and channel the [chromosome].fa/.fai/.dict, padded BEDs, and the four indexed VCFs out as the per-simulation reference bundle under dna_simulations/[simulation_id]/references/. Per the SUBVAR update, keep variant outputs in the reference resource format rather than ClinVar's format.

## Related tools

- **Nextflow** (Workflow engine implementing the SUBSET_REFERENCES_TO_TARGET subworkflow in DSL2) — https://www.nextflow.io/
- **Docker** (Container system providing reproducible environments for the subsetting processes) — https://www.docker.com/
- **samtools** (faidx indexing and chromosome-level FASTA subsetting)
- **bcftools** (Region-aware subsetting of known-variant VCFs with `bcftools view -R`)
- **GATK** (Sequence dictionary creation and SelectVariants-based VCF subsetting to padded target regions)

## Examples

```
nextflow run eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- file_exists / file_format_is checks pass for *_gnomad.vcf.gz, *_mills.vcf.gz, *_1000g.vcf.gz and *_dbsnp.vcf.gz
- Every [simulation_id]_* VCF has a matching .tbi tabix index
- Subset chromosome FASTA is accompanied by both a .fai and a .dict companion file
- Padded target BED files exist alongside the four subset VCFs in dna_simulations/[simulation_id]/references/
- Subworkflow outputs are in reference resource format, not ClinVar format

## Limitations

- Analysis is chromosome-specific (typically chr22) to reduce computational requirements, so whole-genome or multi-chromosome targets are out of scope
- DNA simulations assume capture-based (exome/targeted) sequencing, so reference subsetting depends on a capture BED and padding of captured intervals

## Evidence

- [intro] Reference Preparation: Subsets the reference genome and annotation to your target chromosome and capture regions.: "**Reference Preparation**: Subsets the reference genome and annotation to your target chromosome and capture regions."
- [readme] The pipeline focuses on single chromosomes (typically chr22) to reduce computational requirements while maintaining biological realism.: [evidence span withheld — anchor status: not_found]
- [readme] For DNA simulations, the pipeline assumes exome or targeted sequencing using capture regions.: [evidence span withheld — anchor status: not_found]
- [discussion] Variant outputs now follow the reference format rather than ClinVar’s; SUBSET_REFERENCES_TO_TARGET was updated accordingly.: "variant outputs now follow the reference format rather than ClinVar’s. The SUBSET_REFERENCES_TO_TARGET subworkflow was updated accordingly"
- [results] The DNA reference bundle emitted under dna_simulations/[simulation_id]/references/ contains known variant databases subset to the target regions.: [evidence span withheld — anchor status: not_found]
