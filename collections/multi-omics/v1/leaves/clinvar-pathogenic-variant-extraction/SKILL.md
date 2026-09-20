---
name: clinvar-pathogenic-variant-extraction
description: Use when you need to source the pathogenic variants that will be injected
  into simulated DNA reads — i.e. when defining which ClinVar pathogenic variants
  fall within a target chromosome's capture regions before read simulation.
when_to_use_negative:
- You need variants genome-wide rather than within a single chromosome's capture regions
  — eduomics deliberately focuses on single-chromosome, capture-based (exome/targeted)
  simulation
- You need to preserve ClinVar's native chromosome/position/allele representation
  — the module intentionally recodes outputs to the reference genome format
- The goal is RNA-seq differential expression simulation, which selects genes via
  GO annotation instead of ClinVar variants
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: Nextflow
  role: Workflow engine that executes the SUBVAR extraction process within the nf-core/eduomics
    pipeline
  repo: https://github.com/lescai-teaching/eduomics
- name: Docker
  role: Container engine providing the reproducible environment in which SUBVAR runs
  repo: https://github.com/lescai-teaching/eduomics
- name: Singularity
  role: Alternative container engine for running SUBVAR reproducibly
  repo: https://github.com/lescai-teaching/eduomics
- name: Conda
  role: Alternative environment system for running SUBVAR reproducibly
  repo: https://github.com/lescai-teaching/eduomics
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:0f51941b96e5cee4bb5d12a1b85cb9517f685cabcbcf1a0d45d746c2e962236e
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# clinvar-pathogenic-variant-extraction

## Summary

Extract pathogenic ClinVar variants overlapping user-supplied target capture regions on a single target chromosome and recode them to the reference genome format (GATK.GRCh38), producing bgzipped, tabix-indexed VCFs that the eduomics pipeline injects into simulated reads via SimuSCoP.

## When to use

Use when you need to source the pathogenic variants that will be injected into simulated DNA reads — i.e. when defining which ClinVar pathogenic variants fall within a target chromosome's capture regions before read simulation.

## When NOT to use

- You need variants genome-wide rather than within a single chromosome's capture regions — eduomics deliberately focuses on single-chromosome, capture-based (exome/targeted) simulation
- You need to preserve ClinVar's native chromosome/position/allele representation — the module intentionally recodes outputs to the reference genome format
- The goal is RNA-seq differential expression simulation, which selects genes via GO annotation instead of ClinVar variants

## Inputs

- ClinVar database records
- User-supplied target capture regions (BED) for the target chromosome
- GATK.GRCh38 reference genome configuration (via --genome)
- Nextflow samplesheet row defining the DNA simulation (id, chromosome, coverage, capture)

## Outputs

- bgzip-compressed VCF of pathogenic ClinVar variants harmonised to the reference format ([simulation_id]_*.vcf.gz)
- Tabix index for the variant VCF (.tbi)
- Variant set used for injection into simulated paired-end FASTQ reads and for downstream variant-calling validation

## How to apply

Run the SUBVAR module as a Nextflow process inside the eduomics pipeline under a container engine (Docker, Singularity, or Conda) with the GATK.GRCh38 reference genome configured. Load the ClinVar database and filter records to pathogenic variants that overlap the user-supplied target capture regions on the target chromosome (the pipeline's default focus is a single chromosome, typically chr22). Recode the retained records so alleles, positions and chromosome naming follow the reference genome format rather than ClinVar's native representation — a change made alongside the SUBSET_REFERENCES_TO_TARGET subworkflow update. Compress the harmonised variants with bgzip and create a tabix index following the pipeline's naming convention (e.g. [simulation_id]_clinvar.vcf.gz plus .tbi, matching the gnomAD/Mills/1000G/dbSNP outputs). The extracted variants feed downstream SimuSCoP 2.0.1 read simulation, so they must be loadable by tabix and recognisable by the variant-calling validation step.

## Related tools

- **Nextflow** (Workflow engine that executes the SUBVAR extraction process within the nf-core/eduomics pipeline) — https://github.com/lescai-teaching/eduomics
- **Docker** (Container engine providing the reproducible environment in which SUBVAR runs) — https://github.com/lescai-teaching/eduomics
- **Singularity** (Alternative container engine for running SUBVAR reproducibly) — https://github.com/lescai-teaching/eduomics
- **Conda** (Alternative environment system for running SUBVAR reproducibly) — https://github.com/lescai-teaching/eduomics

## Examples

```
nextflow run eduomics -profile docker --input dna_samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- The retained records are all pathogenic ClinVar variants overlapping the target capture regions (no benign/uncertain-significance records remain)
- Chromosome naming, positions and alleles in the output follow the GATK.GRCh38 reference format rather than ClinVar's native representation
- The .vcf.gz/.tbi pair exists, is loadable with tabix, and follows the [simulation_id]_*.vcf.gz naming convention
- Every simulation that uses these variants subsequently passes the variant-calling validation — the injected pathogenic variant is detected in the disease sample's reads and absent from the normal sample's reads; simulations failing this check are excluded from the final output

## Limitations

- Restricted to pathogenic ClinVar variants within the pipeline's target regions; variants outside the capture regions or on other chromosomes are not considered
- ClinVar's native representation is discarded in favour of the reference genome format, so output coordinates/alleles will not match raw ClinVar records
- Validation is variant-calling based: any extracted variant that cannot be verified as present in the disease sample and absent from the normal sample causes the simulation to be excluded from the final output

## Evidence

- [intro] Variant selection from ClinVar: "Extracts pathogenic variants from ClinVar database within your target regions."
- [discussion] SUBVAR recoded to reference format: "variant outputs now follow the reference format rather than ClinVar's. The SUBSET_REFERENCES_TO_TARGET subworkflow was updated accordingly"
- [readme] DNA simulation mode injects known pathogenic variants: [evidence span withheld — anchor status: not_found]
- [intro] Validation gate on injected variants: "Performs variant calling to verify that injected variants can be detected."
