---
name: genomic-interval-overlap-filtering
description: Use when a variant database such as ClinVar must be subset to exome/target
  capture regions on a specific chromosome and its output harmonised to the reference
  genome format before simulation or calling.
when_to_use_negative:
- The variant track already follows the reference genome format and is already restricted
  to the target capture regions
- The goal is RNA-seq differential-expression simulation, which uses GO-annotation
  gene selection rather than interval overlap of variant databases
- Whole-genome, non-capture analyses where subsetting to capture regions would discard
  intended variants
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Nextflow
  role: Workflow engine executing the SUBVAR filtering process inside the eduomics
    pipeline
  repo: https://github.com/lescai-teaching/eduomics
- name: Docker
  role: Container engine providing a reproducible environment for the SUBVAR process
- name: Singularity
  role: Alternative container engine for running the SUBVAR process
- name: Conda
  role: Alternative environment manager for running the SUBVAR process
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:e35cf015a799773e70e778e037fcf377f8aaced3b9703aed50d186eeb654c0bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lescai-teaching/eduomics
  tools:
  - Conda
  - Docker
  - Nextflow
  - Singularity
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# genomic-interval-overlap-filtering

## Summary

Filters variant records (e.g. pathogenic ClinVar variants) to those overlapping user-supplied target chromosome/capture intervals, then recodes the retained records to match a reference genome naming convention for downstream simulation.

## When to use

Use when a variant database such as ClinVar must be subset to exome/target capture regions on a specific chromosome and its output harmonised to the reference genome format before simulation or calling.

## When NOT to use

- The variant track already follows the reference genome format and is already restricted to the target capture regions
- The goal is RNA-seq differential-expression simulation, which uses GO-annotation gene selection rather than interval overlap of variant databases
- Whole-genome, non-capture analyses where subsetting to capture regions would discard intended variants

## Inputs

- ClinVar database records
- Target capture regions (BED-style capture regions supplied via the samplesheet, e.g. capture.bed)
- Target chromosome specification (e.g. chr22 in samplesheet)
- GATK.GRCh38 reference genome configuration

## Outputs

- bgzip-compressed, reference-format variant VCF restricted to target regions
- Tabix index (.tbi) paired with the .vcf.gz

## How to apply

Run the filtering as a Nextflow process (the eduomics SUBVAR module) under Docker, Singularity, or Conda with the GATK.GRCh38 reference configured. Load the ClinVar database and retain only records flagged as pathogenic that overlap the user-supplied target capture regions on the target chromosome (e.g. chr22). Recode retained records so alleles, positions, and chromosome naming follow the reference genome format rather than ClinVar's native representation, consistent with the SUBSET_REFERENCES_TO_TARGET subworkflow. bgzip-compress the harmonised records and build a tabix index, using the eduomics variant-output naming convention (e.g. [simulation_id]_gnomad.vcf.gz with a .tbi). The rationale is that downstream read simulation (SimuSCoP) and variant-calling validation require intervals expressed in the reference coordinate system.

## Related tools

- **Nextflow** (Workflow engine executing the SUBVAR filtering process inside the eduomics pipeline) — https://github.com/lescai-teaching/eduomics
- **Docker** (Container engine providing a reproducible environment for the SUBVAR process)
- **Singularity** (Alternative container engine for running the SUBVAR process)
- **Conda** (Alternative environment manager for running the SUBVAR process)

## Examples

```
nextflow run eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- Emitted variant output follows the reference genome format rather than ClinVar's native representation
- The .vcf.gz/.tbi pair exists and is loadable for downstream variant-simulation and calling steps
- Retained records are pathogenic and overlap the supplied target capture regions on the target chromosome
- Output naming matches the eduomics convention (e.g. [simulation_id]_gnomad.vcf.gz with .tbi)
- Only simulations passing the downstream variant-calling validation are included in the final output

## Limitations

- Analysis is chromosome-specific (typically chr22) and capture-based, so variants outside the supplied target regions are discarded
- Records must be recoded from ClinVar's representation to the reference format; mismatches break downstream simulation and validation
- Designed for human data with standard reference genomes (GATK.GRCh38 recommended)

## Evidence

- [intro] Variant selection from ClinVar: "Extracts pathogenic variants from ClinVar database within your target regions."
- [discussion] SUBVAR harmonisation: "variant outputs now follow the reference format rather than ClinVar's. The SUBSET_REFERENCES_TO_TARGET subworkflow was updated accordingly"
- [readme] README DNA branch: [evidence span withheld — anchor status: not_found]
- [readme] README simulation modes: [evidence span withheld — anchor status: not_found]
