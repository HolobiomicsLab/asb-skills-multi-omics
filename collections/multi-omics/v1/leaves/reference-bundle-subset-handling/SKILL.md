---
name: reference-bundle-subset-handling
description: Use when a DNA simulation task defines a single chromosome (e.g. chr22)
  and capture regions, and the GATK.GRCh38 reference plus annotation must be reduced
  to a chromosome-specific bundle before variant extraction or read simulation.
when_to_use_negative:
- When the teaching goal is whole-genome or multi-chromosome analysis — the pipeline
  explicitly assumes chromosome-specific, capture-based analysis to stay computationally
  tractable
- When variants must remain in ClinVar's own format — SUBVAR outputs are harmonised
  to follow the reference format instead of ClinVar's
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Nextflow
  role: Workflow engine executing the SUBSET_REFERENCES_TO_TARGET subworkflow that
    trims the reference genome and annotation to target regions
  repo: https://github.com/lescai-teaching/eduomics
- name: GATK.GRCh38
  role: Recommended source reference genome configuration from which the chromosome/capture
    subset is derived
- name: Docker
  role: Container engine profile used to run the reference-preparation steps reproducibly
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:90f14a412f6590a937f7a68e9773e6d44ce1cbb66422aa7a19d90072dab0c583
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lescai-teaching/eduomics
  tools:
  - Conda
  - Docker
  - FastQC
  - GATK.GRCh38
  - MultiQC
  - Nextflow
  - Singularity
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# reference-bundle-subset-handling

## Summary

Subsets a full human reference genome and annotation (GATK.GRCh38 recommended) to a target chromosome and capture regions so downstream DNA simulation, variant injection, and variant calling operate on a compact, consistent reference bundle.

## When to use

Use when a DNA simulation task defines a single chromosome (e.g. chr22) and capture regions, and the GATK.GRCh38 reference plus annotation must be reduced to a chromosome-specific bundle before variant extraction or read simulation.

## When NOT to use

- When the teaching goal is whole-genome or multi-chromosome analysis — the pipeline explicitly assumes chromosome-specific, capture-based analysis to stay computationally tractable
- When variants must remain in ClinVar's own format — SUBVAR outputs are harmonised to follow the reference format instead of ClinVar's

## Inputs

- DNA simulation samplesheet rows with columns id,type,chromosome,coverage,capture,reps,groups,simthreshold (e.g. dna_simulation_1,dna,chr22,100,https://example.com/capture.bed,1,2,0.3)
- Capture regions BED file for targeted/exome sequencing regions
- Full reference genome and annotation configured as GATK.GRCh38

## Outputs

- Subset reference bundle trimmed to the target chromosome and capture regions
- Simulation-specific variant reference files: [simulation_id]_gnomad.vcf.gz(.tbi), [simulation_id]_mills.vcf.gz(.tbi), [simulation_id]_1000g.vcf.gz(.tbi), [simulation_id]_dbsnp.vcf.gz(.tbi) in reference format rather than ClinVar format
- Complete reference package shipped with each simulation for downstream student analyses

## How to apply

Run the eduomics pipeline (Nextflow ≥25.04.3, via Docker, Singularity, or Conda) with a samplesheet whose DNA rows specify the target chromosome, coverage, and a capture BED (e.g. dna_simulation_1,dna,chr22,100,https://example.com/capture.bed,1,2,0.3) and pass --genome GATK.GRCh38. The SUBSET_REFERENCES_TO_TARGET subworkflow trims the reference genome and annotation to the target chromosome and capture regions, reducing computational requirements while maintaining biological realism (the README recommends focusing on single chromosomes such as chr22). The subset references are also harmonised with the variant resources: SUBVAR variant outputs follow the reference format rather than ClinVar's, so the subset bundle and the extracted ClinVar/gnomAD/Mills/1000 Genomes/dbSNP VCFs stay coordinate- and representation-consistent. The resulting subset bundle then feeds SimuSCoP read simulation with injected variants and the subsequent variant-calling validation step.

## Related tools

- **Nextflow** (Workflow engine executing the SUBSET_REFERENCES_TO_TARGET subworkflow that trims the reference genome and annotation to target regions) — https://github.com/lescai-teaching/eduomics
- **GATK.GRCh38** (Recommended source reference genome configuration from which the chromosome/capture subset is derived)
- **Docker** (Container engine profile used to run the reference-preparation steps reproducibly)

## Examples

```
nextflow run eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- Subset bundle contains reference genome and annotation restricted to the samplesheet's target chromosome (e.g. chr22) and capture BED regions
- Subset references are consistent with the SUBVAR variant outputs, which follow the reference format rather than ClinVar's format
- Per-simulation VCFs (gnomAD, Mills, 1000g, dbSNP) are emitted with .vcf.gz and .tbi companions in the simulation's output directory
- Only simulations passing downstream variant-detection validation are included in the final output, implying the subset bundle supported successful variant calling

## Limitations

- Chromosome-specific scope: the pipeline focuses on single chromosomes (typically chr22) to reduce computational requirements, so the subset bundle is not suited for genome-wide analyses
- Capture-based assumption: for DNA simulations the pipeline assumes exome or targeted sequencing using capture regions; undefined capture regions undermine the subsetting
- Variant representation changes: SUBVAR outputs now follow the reference format rather than ClinVar's, so consumers expecting ClinVar-native formatting must adapt

## Evidence

- [intro] intro: reference preparation step: "**Reference Preparation**: Subsets the reference genome and annotation to your target chromosome and capture regions."
- [discussion] discussion: SUBSET_REFERENCES_TO_TARGET harmonisation: "variant outputs now follow the reference format rather than ClinVar’s. The SUBSET_REFERENCES_TO_TARGET subworkflow was updated accordingly"
- [readme] readme: chromosome-specific rationale: [evidence span withheld — anchor status: not_found]
- [intro] intro: recommended reference: "**Reference genome configured** (we recommend using `GATK.GRCh38`)"
