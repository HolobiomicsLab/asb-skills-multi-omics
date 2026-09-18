---
name: vcf-compression-and-tabix-indexing
description: Use when variant records sourced from ClinVar must be harmonised to GATK.GRCh38
  reference naming (alleles, positions, chromosome), bgzip-compressed and tabix-indexed
  for downstream read simulation and variant calling.
when_to_use_negative:
- Variants are already in reference-genome format and only need reformatting to a
  different schema (e.g. BED or plain text annotations)
- The analysis targets whole-genome rather than chromosome-specific capture-region
  simulation, where SUBVAR's region-restricted ClinVar extraction does not apply
- Input is raw sequencing data (FASTQ) rather than variant records to be indexed
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Nextflow
  role: Runs the SUBVAR module as a workflow process inside the eduomics pipeline
    with the GATK.GRCh38 genome configured
  repo: https://github.com/lescai-teaching/eduomics
- name: Docker
  role: Container engine option for executing the eduomics/SUBVAR process reproducibly
- name: Singularity
  role: Container engine option for executing the eduomics/SUBVAR process reproducibly
- name: Conda
  role: Alternative environment manager for executing the eduomics/SUBVAR process
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:fee8ce45cf14a7d31fbf4856660286f466a2c6376972adbf4faf1727d41bff2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lescai-teaching/eduomics
  tools:
  - Conda
  - Docker
  - Nextflow
  - Singularity
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# vcf-compression-and-tabix-indexing

## Summary

In the eduomics DNA simulation workflow, pathogenic variants extracted from ClinVar (SUBVAR) are recoded to the reference genome format, bgzip-compressed, and tabix-indexed so they are delivered as .vcf.gz/.tbi pairs matching the pipeline's other supplied variant tracks (e.g. [simulation_id]_gnomad.vcf.gz).

## When to use

Use when variant records sourced from ClinVar must be harmonised to GATK.GRCh38 reference naming (alleles, positions, chromosome), bgzip-compressed and tabix-indexed for downstream read simulation and variant calling.

## When NOT to use

- Variants are already in reference-genome format and only need reformatting to a different schema (e.g. BED or plain text annotations)
- The analysis targets whole-genome rather than chromosome-specific capture-region simulation, where SUBVAR's region-restricted ClinVar extraction does not apply
- Input is raw sequencing data (FASTQ) rather than variant records to be indexed

## Inputs

- ClinVar database records (pathogenic variants)
- User-supplied target capture region BED file
- Target chromosome definition (e.g. chr22) from the samplesheet
- GATK.GRCh38 reference genome

## Outputs

- bgzip-compressed, reference-harmonised variant VCF (.vcf.gz)
- Tabix index file (.tbi)
- Variant tracks named per eduomics convention (e.g. [simulation_id]_gnomad.vcf.gz / .vcf.gz.tbi)

## How to apply

Run the SUBVAR module as a Nextflow process (Docker, Singularity, or Conda container) with the GATK.GRCh38 reference genome configured. Filter ClinVar to pathogenic variants overlapping the user-supplied target capture regions on the target chromosome (typically chr22). Recode the retained records so alleles, positions and chromosome naming follow the reference genome format rather than ClinVar's native representation, consistent with the SUBSET_REFERENCES_TO_TARGET subworkflow update. Then bgzip-compress the harmonised records and create a tabix index, naming the output pair according to the eduomics convention used for other supplied variant tracks such as [simulation_id]_gnomad.vcf.gz with its .tbi.

## Related tools

- **Nextflow** (Runs the SUBVAR module as a workflow process inside the eduomics pipeline with the GATK.GRCh38 genome configured) — https://github.com/lescai-teaching/eduomics
- **Docker** (Container engine option for executing the eduomics/SUBVAR process reproducibly)
- **Singularity** (Container engine option for executing the eduomics/SUBVAR process reproducibly)
- **Conda** (Alternative environment manager for executing the eduomics/SUBVAR process)

## Examples

```
nextflow run eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- The .vcf.gz/.tbi pair exists for every emitted variant track and the index is loadable (e.g. tabix-queryable) for downstream variant-simulation and calling steps
- Alleles, positions and chromosome naming in the output VCF follow the GATK.GRCh38 reference format rather than ClinVar's native representation
- Retained records are pathogenic ClinVar variants overlapping the user-supplied target capture regions on the target chromosome
- Output filenames follow the eduomics convention ([simulation_id]_*.vcf.gz with matching .tbi) used for other supplied variant tracks

## Limitations

- Reference-format harmonisation was a recent update (PR-linked change), so earlier pipeline versions emitted variants in ClinVar's format
- SUBVAR extraction is restricted to pathogenic variants within the supplied capture regions on a single chromosome, so it cannot inject variants outside those regions
- Only simulations passing validation are included in the final output, so an unloadable or malformed index will drop the simulation

## Evidence

- [intro] Extracts pathogenic variants from ClinVar database within your target regions.: "Extracts pathogenic variants from ClinVar database within your target regions."
- [discussion] variant outputs now follow the reference format rather than ClinVar's. The SUBSET_REFERENCES_TO_TARGET subworkflow was updated accordingly: "variant outputs now follow the reference format rather than ClinVar’s. The SUBSET_REFERENCES_TO_TARGET subworkflow was updated accordingly"
- [intro] **`[simulation_id]_gnomad.vcf.gz`** and **`[simulation_id]_gnomad.vcf.gz.tbi`**: gnomAD variants: "**`[simulation_id]_gnomad.vcf.gz`** and **`[simulation_id]_gnomad.vcf.gz.tbi`**: gnomAD variants"
- [intro] Subsets the reference genome and annotation to your target chromosome and capture regions.: "Subsets the reference genome and annotation to your target chromosome and capture regions."
