---
name: workflow-metadata-annotation
description: Use when a Nextflow module must extract a gene annotation from a simulated
  variant record in reference format and pass it forward as workflow meta to condition
  a downstream AI-generated scenario.
when_to_use_negative:
- The variant record is still in ClinVar format rather than the harmonised reference
  format — parsing must target the reference-format fields (SUBVAR outputs were updated
  accordingly).
- The goal is RNA-seq simulation metadata (gene selection is driven by GO annotation
  there, not by injected pathogenic variant records).
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Nextflow
  role: Workflow engine in which the PYCONVERTSIM process module is scaffolded following
    the nf-core module structure
  repo: https://github.com/lescai-teaching/eduomics
- name: Docker
  role: Container system used to containerise the PYCONVERTSIM module per nf-core
    one-container-per-process conventions
  repo: https://github.com/lescai-teaching/eduomics
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:1c19bd3208a8923ee265606951752c6ea33fe8a9c2a84da6b12efd2d80ac39e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lescai-teaching/eduomics
  tools:
  - Docker
  - Nextflow
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2857142857142857
n_claims: 7
---

# workflow-metadata-annotation

## Summary

Reconstructs the eduomics PYCONVERTSIM Nextflow module that parses a simulated variant record (post-SUBVAR, in reference format) to extract the gene name of the injected pathogenic variant, emits it as gene_annotation.tsv, and propagates it into the variant-calling workflow meta and the AI educational scenario prompt.

## When to use

Use when a Nextflow module must extract a gene annotation from a simulated variant record in reference format and pass it forward as workflow meta to condition a downstream AI-generated scenario.

## When NOT to use

- The variant record is still in ClinVar format rather than the harmonised reference format — parsing must target the reference-format fields (SUBVAR outputs were updated accordingly).
- The goal is RNA-seq simulation metadata (gene selection is driven by GO annotation there, not by injected pathogenic variant records).

## Inputs

- Simulated variant record from the eduomics DNA simulation subworkflow (reference-format VCF-style record, post-SUBVAR harmonisation)
- nf-core/eduomics module scaffolding (Nextflow DSL2 process template, meta maps)
- Container configuration (Docker)

## Outputs

- gene_annotation.tsv containing the extracted gene name of the input variant record
- Gene name propagated into the variant-calling workflow meta map
- Gene name used as the condition for the AI educational scenario prompt ([simulation_id]_scenario.txt)

## How to apply

Scaffold a Nextflow process module named PYCONVERTSIM following the nf-core/eduomics module structure, containerised with Docker (consistent with the pipeline's one-container-per-process DSL2 design). Load the simulated variant record produced by the DNA simulation subworkflow, which after harmonisation follows the reference genome format rather than ClinVar's format, so parse fields accordingly. Extract the gene name associated with the simulated pathogenic variant and emit it as gene_annotation.tsv. Emit the gene name into the variant-calling workflow meta and pass the same gene name as the condition for the AI scenario prompt, since each DNA simulation outputs a [simulation_id]_scenario.txt file whose biological context is conditioned on that gene.

## Related tools

- **Nextflow** (Workflow engine in which the PYCONVERTSIM process module is scaffolded following the nf-core module structure) — https://github.com/lescai-teaching/eduomics
- **Docker** (Container system used to containerise the PYCONVERTSIM module per nf-core one-container-per-process conventions) — https://github.com/lescai-teaching/eduomics

## Examples

```
nextflow run nf-core/eduomics -profile docker --input dna_samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- gene_annotation.tsv contains exactly the gene name of the input variant record parsed in reference format
- The extracted gene matches the gene of the injected pathogenic variant used to teach the variant calling workflow
- The gene name appears in the variant-calling workflow meta map
- The same gene name conditions the AI scenario prompt, and the resulting [simulation_id]_scenario.txt provides biological context for that gene
- nf-test / linting (GitHub Actions CI and linting workflows) passes for the new module

## Limitations

- The module depends on SUBVAR's harmonisation: variant outputs follow the reference format rather than ClinVar's, so field parsing breaks if older ClinVar-format records are supplied.
- The pipeline focuses on single chromosomes (typically chr22) with capture-based, paired-end simulation, so the injected variant — and hence the extracted gene — is limited to the subset target regions.

## Evidence

- [intro] Each DNA simulation outputs a [simulation_id]_scenario.txt file, which is an AI-generated educational scenario providing biological context for the simulation.: [evidence span withheld — anchor status: not_found]
- [discussion] Variant outputs harmonised to reference format.: "variant outputs now follow the reference format rather than ClinVar's. The SUBSET_REFERENCES_TO_TARGET subworkflow was updated accordingly"
- [readme] README educational context statement.: [evidence span withheld — anchor status: not_found]
- [readme] README DNA simulation logic.: [evidence span withheld — anchor status: not_found]
