---
name: variant-record-harmonisation-to-reference-format
description: 'Use when preparing injected-variant tracks for a DNA simulation pipeline:
  source variants are in ClinVar''s native format but downstream simulation and variant-calling
  steps require reference-genome-formatted, bgzipped, tabix-indexed VCFs.'
when_to_use_negative:
- Variants are already supplied in reference-genome format as bgzipped, tabix-indexed
  VCFs (e.g. the gnomAD, Mills, 1000 Genomes, or dbSNP tracks already emitted by the
  pipeline)
- The goal is RNA-seq differential expression simulation, where gene selection relies
  on GO annotation rather than ClinVar variant extraction
- Analysis requires population-frequency annotation rather than pathogenic-variant
  extraction from ClinVar
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3577
tools:
- name: Nextflow
  role: Workflow engine running SUBVAR as a Nextflow process inside the eduomics pipeline
  repo: https://github.com/lescai-teaching/eduomics
- name: Docker
  role: Container engine option for executing the SUBVAR process reproducibly
- name: Singularity
  role: Container engine option for executing the SUBVAR process reproducibly
- name: Conda
  role: Container/environment system option for executing the SUBVAR process
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:16e683b406180fbff3e4e59c63af6f9517148a6b5df7682f52e037a69b8caf66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lescai-teaching/eduomics
  tools:
  - Conda
  - Docker
  - Nextflow
  - Singularity
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# variant-record-harmonisation-to-reference-format

## Summary

Extracts pathogenic variants from ClinVar restricted to target capture regions and recodes alleles, positions, and chromosome naming to the reference genome format (e.g. GATK.GRCh38) instead of ClinVar's native representation, emitting bgzip-compressed VCF with a tabix index ready for variant simulation.

## When to use

Use when preparing injected-variant tracks for a DNA simulation pipeline: source variants are in ClinVar's native format but downstream simulation and variant-calling steps require reference-genome-formatted, bgzipped, tabix-indexed VCFs.

## When NOT to use

- Variants are already supplied in reference-genome format as bgzipped, tabix-indexed VCFs (e.g. the gnomAD, Mills, 1000 Genomes, or dbSNP tracks already emitted by the pipeline)
- The goal is RNA-seq differential expression simulation, where gene selection relies on GO annotation rather than ClinVar variant extraction
- Analysis requires population-frequency annotation rather than pathogenic-variant extraction from ClinVar

## Inputs

- ClinVar database records (pathogenic variants)
- User-supplied target capture regions (BED, e.g. capture.bed)
- Target chromosome designation from the samplesheet (e.g. chr22)
- Reference genome configured as GATK.GRCh38

## Outputs

- bgzip-compressed harmonised pathogenic-variant VCF ([simulation_id]_clinv.vcf.gz style, following eduomics naming convention)
- Tabix index (.tbi) for the compressed VCF
- Variant track loadable for downstream SimuSCoP read simulation and variant-calling validation

## How to apply

Run the SUBVAR module as a Nextflow process inside the eduomics pipeline via a container engine (Docker, Singularity, or Conda) with the GATK.GRCh38 reference genome configured. Load the ClinVar database and filter records to pathogenic variants overlapping the user-supplied target capture regions on the target chromosome (e.g. chr22). Recode retained records so alleles, positions and chromosome naming follow the reference genome format rather than ClinVar's native representation, consistent with the SUBSET_REFERENCES_TO_TARGET subworkflow update. Then bgzip-compress the harmonised records and create a tabix index, following the eduomics variant-output naming convention used for the other supplied tracks (e.g. [simulation_id]_gnomad.vcf.gz with a .tbi).

## Related tools

- **Nextflow** (Workflow engine running SUBVAR as a Nextflow process inside the eduomics pipeline) — https://github.com/lescai-teaching/eduomics
- **Docker** (Container engine option for executing the SUBVAR process reproducibly)
- **Singularity** (Container engine option for executing the SUBVAR process reproducibly)
- **Conda** (Container/environment system option for executing the SUBVAR process)

## Examples

```
nextflow run eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- Emitted variant output follows the reference format rather than ClinVar's native representation
- The .vcf.gz/.tbi pair exists and is loadable (tabix index valid)
- Retained records are pathogenic ClinVar variants overlapping the supplied target capture regions on the target chromosome
- Output naming matches the eduomics convention used for other supplied variant tracks (e.g. [simulation_id]_gnomad.vcf.gz with a .tbi)
- Downstream, the injected pathogenic variant is detected in the 'disease' sample reads and absent from the 'normal' sample during variant-calling validation

## Limitations

- Analysis is chromosome-specific by design (typically chr22) and capture-based; whole-genome or non-targeted use is outside the pipeline's assumptions
- Harmonisation is tied to a specific reference genome configuration (GATK.GRCh38 recommended); other references require re-running the recoding
- Only pathogenic ClinVar variants within target regions are extracted — variants outside capture regions or of other clinical significance are excluded

## Evidence

- [intro] Extracts pathogenic variants from ClinVar database within your target regions.: "Extracts pathogenic variants from ClinVar database within your target regions."
- [discussion] variant outputs now follow the reference format rather than ClinVar's: "variant outputs now follow the reference format rather than ClinVar’s. The SUBSET_REFERENCES_TO_TARGET subworkflow was updated accordingly"
- [readme] capture BED from samplesheet: [evidence span withheld — anchor status: not_found]
- [intro] gnomAD naming convention: "**`[simulation_id]_gnomad.vcf.gz`** and **`[simulation_id]_gnomad.vcf.gz.tbi`**: gnomAD variants"
