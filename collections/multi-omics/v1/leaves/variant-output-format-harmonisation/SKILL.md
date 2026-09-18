---
name: variant-output-format-harmonisation
description: Use when a Nextflow module must extract the gene name from a simulated
  variant record in reference (not ClinVar) format so it can populate workflow meta
  and condition an AI scenario prompt in storyline-based learning.
when_to_use_negative:
- The variant record is still in ClinVar format rather than the harmonised reference
  format — harmonisation (SUBVAR/SUBSET_REFERENCES_TO_TARGET) must happen first
- The task targets RNA-seq differential expression simulation outputs, which do not
  carry pathogenic variant records
- Real clinical variant annotation is required — eduomics variants are simulated educational
  data, not diagnostics
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Nextflow
  role: Workflow engine used to implement the PYCONVERTSIM process module following
    nf-core module structure
  repo: https://github.com/lescai-teaching/eduomics
- name: Docker
  role: Containerisation of the PYCONVERTSIM process, one container per process in
    the DSL2 pipeline
  repo: https://github.com/lescai-teaching/eduomics
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:6bdc06910aea863f7742e3eb09837d52582d3cba545d0947d228ce5b9e556441
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lescai-teaching/eduomics
  tools:
  - Docker
  - Nextflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# variant-output-format-harmonisation

## Summary

In the eduomics pipeline, this skill covers reconstructing the PYCONVERTSIM Nextflow process that parses the simulated DNA variant record (harmonised to reference format rather than ClinVar's) to extract the gene name of the injected pathogenic variant, which then conditions the AI-generated educational scenario.

## When to use

Use when a Nextflow module must extract the gene name from a simulated variant record in reference (not ClinVar) format so it can populate workflow meta and condition an AI scenario prompt in storyline-based learning.

## When NOT to use

- The variant record is still in ClinVar format rather than the harmonised reference format — harmonisation (SUBVAR/SUBSET_REFERENCES_TO_TARGET) must happen first
- The task targets RNA-seq differential expression simulation outputs, which do not carry pathogenic variant records
- Real clinical variant annotation is required — eduomics variants are simulated educational data, not diagnostics

## Inputs

- Simulated DNA variant record in reference format (output of the eduomics DNA simulation subworkflow, derived from a ClinVar-extracted pathogenic variant)
- Samplesheet row (id,type,chromosome,coverage,capture,...) defining the DNA simulation

## Outputs

- gene_annotation.tsv containing the gene name of the input variant record in reference format
- Variant-calling workflow meta populated with the extracted gene name
- Gene name used as the condition for the AI scenario prompt ([simulation_id]_scenario.txt downstream)

## How to apply

Scaffold a Nextflow process module named PYCONVERTSIM following the nf-core/eduomics module structure, containerised with Docker (one container per process per the DSL2 implementation). Load the variant record emitted by the DNA simulation subworkflow, noting that SUBVAR outputs are now harmonised to the reference format rather than ClinVar's, with SUBSET_REFERENCES_TO_TARGET updated accordingly. Parse the variant record fields to extract the gene name associated with the simulated pathogenic variant and emit it as gene_annotation.tsv. Pass the same gene name forward as the condition for the AI scenario prompt so that [simulation_id]_scenario.txt provides biological context matching the injected variant.

## Related tools

- **Nextflow** (Workflow engine used to implement the PYCONVERTSIM process module following nf-core module structure) — https://github.com/lescai-teaching/eduomics
- **Docker** (Containerisation of the PYCONVERTSIM process, one container per process in the DSL2 pipeline) — https://github.com/lescai-teaching/eduomics

## Examples

```
nextflow run nf-core/eduomics -profile docker --input dna_samplesheet.csv --genome GATK.GRCh38
```

## Evaluation signals

- gene_annotation.tsv contains exactly the gene name of the input variant record in reference format
- The extracted gene matches the gene of the injected pathogenic variant used to teach the variant calling workflow
- The gene name correctly populates the variant-calling workflow meta
- The AI scenario prompt is conditioned on the same gene name, producing a coherent [simulation_id]_scenario.txt
- nf-test unit tests and linting for the module pass in CI

## Limitations

- Variant outputs follow the reference format rather than ClinVar's; older ClinVar-format records will not parse as expected
- Analysis is chromosome-specific (typically chr22) with capture-based exome/targeted simulation, so gene extraction is limited to target regions
- Only simulations passing validation are included in the final output

## Evidence

- [discussion] variant outputs now follow the reference format rather than ClinVar’s. The SUBSET_REFERENCES_TO_TARGET subworkflow was updated accordingly: "variant outputs now follow the reference format rather than ClinVar’s. The SUBSET_REFERENCES_TO_TARGET subworkflow was updated accordingly"
- [intro] Each DNA simulation outputs a [simulation_id]_scenario.txt file, which is an AI-generated educational scenario providing biological context for the simulation.: [evidence span withheld — anchor status: not_found]
- [readme] Injects known pathogenic variants into specific genomic regions and generates realistic sequencing reads: [evidence span withheld — anchor status: not_found]
