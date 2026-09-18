---
name: nextflow-process-module-development
description: Use when a new nf-core/eduomics process must be added — e.g. parsing
  a simulated variant record in reference format to extract the gene name of an injected
  pathogenic variant for variant-calling meta.
when_to_use_negative:
- The variant record is still in ClinVar format rather than the reference format emitted
  by the harmonised SUBVAR outputs — parsing ClinVar-style fields would contradict
  the pipeline's harmonisation
- The input is RNA-seq simulation output (count matrices, FASTQ from Polyester) rather
  than a DNA simulation variant record
- A nf-core/modules community module already implements the process — the README directs
  that such processes be installed from nf-core/modules instead of rewritten
edam_topics:
- http://edamontology.org/topic_0768
tools:
- name: Nextflow
  role: DSL2 workflow framework in which the process module (e.g. PYCONVERTSIM) is
    scaffolded and its channels/outputs are defined; version >=25.04.4
  repo: https://www.nextflow.io/
- name: Docker
  role: Container system used to containerise each process module so dependencies
    are reproducible
- name: nf-core modules
  role: Shared module repository from which reusable eduomics processes are submitted
    to and installed from, instead of bespoke reimplementation
  repo: https://github.com/nf-core/modules
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:2515ab622a8ddaf95c6af2936f0281358d3b4f234fb82f37e4cc75e4055096e8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - Nextflow
  - nf-core modules
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0768
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# nextflow-process-module-development

## Summary

Scaffolding a containerised Nextflow DSL2 process module (e.g. PYCONVERTSIM) within the nf-core/eduomics structure so that simulated variant records can be parsed to extract the gene name and emit it for downstream workflow meta and AI scenario conditioning.

## When to use

Use when a new nf-core/eduomics process must be added — e.g. parsing a simulated variant record in reference format to extract the gene name of an injected pathogenic variant for variant-calling meta.

## When NOT to use

- The variant record is still in ClinVar format rather than the reference format emitted by the harmonised SUBVAR outputs — parsing ClinVar-style fields would contradict the pipeline's harmonisation
- The input is RNA-seq simulation output (count matrices, FASTQ from Polyester) rather than a DNA simulation variant record
- A nf-core/modules community module already implements the process — the README directs that such processes be installed from nf-core/modules instead of rewritten

## Inputs

- Simulated variant record from the eduomics DNA simulation subworkflow, in reference format (not ClinVar format)
- nf-core/eduomics module scaffold and pipeline config (samplesheet.csv, genome GATK.GRCh38)
- Docker container image for the module process

## Outputs

- gene_annotation.tsv containing the gene name extracted from the input variant record
- Emitted gene name channel populating the variant-calling workflow meta
- Gene name passed as the condition for the AI-generated educational scenario prompt ([simulation_id]_scenario.txt)

## How to apply

Scaffold a Nextflow process module named after its function (e.g. PYCONVERTSIM) following the nf-core/eduomics module structure, and containerise it with Docker so each DSL2 process uses one container, matching the pipeline's one-container-per-process design. Load the simulated variant record produced by the DNA simulation subworkflow — per the paper's harmonisation these outputs now follow the reference format rather than ClinVar's, so parse reference-format variant fields rather than ClinVar-style HGVS. Parse the record to extract the gene name of the simulated pathogenic variant and emit it (e.g. as gene_annotation.tsv) so it can populate the variant-calling workflow meta. Pass the same gene name forward as the condition for the AI scenario prompt used in storyline-based learning. Validate that the module output contains the gene name of the input variant record and matches the gene of the injected pathogenic variant used to teach the variant calling workflow.

## Related tools

- **Nextflow** (DSL2 workflow framework in which the process module (e.g. PYCONVERTSIM) is scaffolded and its channels/outputs are defined; version >=25.04.4) — https://www.nextflow.io/
- **Docker** (Container system used to containerise each process module so dependencies are reproducible)
- **nf-core modules** (Shared module repository from which reusable eduomics processes are submitted to and installed from, instead of bespoke reimplementation) — https://github.com/nf-core/modules

## Examples

```
nextflow run eduomics -profile docker --input dna_samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- gene_annotation.tsv contains the gene name of the input variant record in reference format
- The extracted gene name matches the gene of the injected pathogenic variant used to teach the variant calling workflow
- The module passes nf-core linting (nf-core pipelines run linting via GitHub Actions workflows/ linting.yml)
- The module runs under 'nextflow run eduomics -profile docker' with the container resolved per process
- The gene name propagates correctly into the variant-calling workflow meta and the AI scenario prompt condition

## Limitations

- Variant outputs were harmonised to follow the reference format rather than ClinVar's, so modules written against ClinVar-format fields (e.g. older SUBVAR output) must be updated, as was done for the SUBSET_REFERENCES_TO_TARGET subworkflow
- The pipeline focuses on single chromosomes (typically chr22) and capture-based (exome/targeted) sequencing, so gene extraction is scoped to those target regions
- Human reference data only, with GATK.GRCh38 recommended as the configured reference genome

## Evidence

- [readme] Containerisation is one-container-per-process in the DSL2 implementation: [evidence span withheld — anchor status: not_found]
- [readme] Reusable processes should come from nf-core/modules: [evidence span withheld — anchor status: not_found]
- [intro] Container system options for module execution: "**Container system** (Docker, Singularity, or Conda)"
- [discussion] Variant outputs harmonised to reference format: "variant outputs now follow the reference format rather than ClinVar’s."
- [intro] Gene name conditioning of the AI scenario: [evidence span withheld — anchor status: not_found]
