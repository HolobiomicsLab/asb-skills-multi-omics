---
name: genomic-interval-bed-processing
description: Use when a simulation or analysis must be restricted to a specific chromosome
  plus exome capture regions, and reference FASTA, .dict/.fai indices, and known-variant
  VCFs (gnomAD, Mills, 1000G, dbSNP) need region-specific subsets.
when_to_use_negative:
- The analysis targets the whole genome rather than a single chromosome plus capture
  regions (eduomics subsets specifically to reduce computational load)
- The variant resources are already region-subset and tabix-indexed for the exact
  target intervals
- The target dataset is transcriptomic (RNA branch subsets transcriptome references,
  not capture-region genomic intervals)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: Nextflow
  role: Workflow engine running the SUBSET_REFERENCES_TO_TARGET subworkflow
  repo: https://github.com/lescai-teaching/eduomics
- name: Docker
  role: Container system providing reproducible environments for the subsetting processes
  repo: https://www.docker.com/
- name: GATK.GRCh38
  role: Recommended reference genome bundle supplying the FASTA to be indexed and
    subset
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:ddfd084e5f3c18a8f6f05cf2fe130c91d709eaaf6ab03c962db13916f9f1a697
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lescai-teaching/eduomics
  tools:
  - Docker
  - GATK.GRCh38
  - Nextflow
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# genomic-interval-bed-processing

## Summary

Subsets reference genome and known-variant VCF resources to a target chromosome and padded exome-capture intervals, emitting per-simulation reference bundles (subset FASTA with .fai/.dict, padded BEDs, bgzipped and tabix-indexed VCFs) as used in the eduomics DNA simulation branch.

## When to use

Use when a simulation or analysis must be restricted to a specific chromosome plus exome capture regions, and reference FASTA, .dict/.fai indices, and known-variant VCFs (gnomAD, Mills, 1000G, dbSNP) need region-specific subsets.

## When NOT to use

- The analysis targets the whole genome rather than a single chromosome plus capture regions (eduomics subsets specifically to reduce computational load)
- The variant resources are already region-subset and tabix-indexed for the exact target intervals
- The target dataset is transcriptomic (RNA branch subsets transcriptome references, not capture-region genomic intervals)

## Inputs

- GATK.GRCh38 reference FASTA (e.g. chr22)
- Exome capture BED file (e.g. from the samplesheet 'capture' column URL)
- Known-variant VCF resources: gnomAD, Mills indels, 1000 Genomes, dbSNP

## Outputs

- [chromosome].fa with .fai and .dict companions
- Padded target-region BED files
- [simulation_id]_gnomad.vcf.gz(+.tbi), [simulation_id]_mills.vcf.gz(+.tbi), [simulation_id]_1000g.vcf.gz(+.tbi), [simulation_id]_dbsnp.vcf.gz(+.tbi)
- Per-simulation reference bundle under dna_simulations/[simulation_id]/references/

## How to apply

Load the GATK.GRCh38 reference FASTA (eduomics' recommended genome) and build the samtools-style .fai index and GATK .dict sequence dictionary. Subset the genome to the target chromosome (e.g. chr22) via samtools faidx or GATK reference subsetting, emitting [chromosome].fa with its .fai and .dict companions. Take the exome capture BED, pad the captured intervals to define final target regions, then subset known-variant resources (gnomAD, Mills indels, 1000 Genomes, dbSNP) to the padded regions with a region-aware VCF tool such as bcftools view -R or GATK SelectVariants, writing bgzipped VCFs per simulation. Tabix-index each subset VCF so every [simulation_id]_* VCF is paired with its .tbi, emit the padded target BED files, and channel the [chromosome].fa/.fai/.dict, padded BEDs, and four subset/indexed VCFs out as the per-simulation reference bundle. Note the SUBVAR harmonisation: variant outputs keep the reference resource format, not ClinVar's format.

## Related tools

- **Nextflow** (Workflow engine running the SUBSET_REFERENCES_TO_TARGET subworkflow) — https://github.com/lescai-teaching/eduomics
- **Docker** (Container system providing reproducible environments for the subsetting processes) — https://www.docker.com/
- **GATK.GRCh38** (Recommended reference genome bundle supplying the FASTA to be indexed and subset)

## Examples

```
bcftools view -R padded_targets.bed -Oz -o ${simulation_id}_gnomad.vcf.gz gnomad.vcf.gz && tabix -p vcf ${simulation_id}_gnomad.vcf.gz
```

## Evaluation signals

- Every [simulation_id]_* known-variant VCF (gnomad, mills, 1000g, dbsnp) has a matching .tbi index (file_exists / file_format_is checks)
- [chromosome].fa, .fai, and .dict are all present in the reference bundle
- Subset VCFs contain only records within the padded capture intervals defined by the BED
- Variant outputs follow the reference resource format, not ClinVar's format (per the SUBSET_REFERENCES_TO_TARGET update)
- Subworkflow outputs are channelled under dna_simulations/[simulation_id]/references/

## Limitations

- Pipeline focuses on single chromosomes (typically chr22) to reduce computational requirements, so whole-genome targets are out of scope
- Assumes capture-based (exome/targeted) sequencing; intervals must be padded before subsetting variant resources
- SUBVAR/SUBSET_REFERENCES_TO_TARGET outputs were harmonised to the reference format rather than ClinVar's, so downstream tools expecting ClinVar-format variants may need adaptation

## Evidence

- [intro] reference prep: "**Reference Preparation**: Subsets the reference genome and annotation to your target chromosome and capture regions."
- [methods] workflow: [evidence span withheld — anchor status: not_found]
- [discussion] format update: "variant outputs now follow the reference format rather than ClinVar’s. The SUBSET_REFERENCES_TO_TARGET subworkflow was updated accordingly"
- [readme] readme scope: [evidence span withheld — anchor status: not_found]
