---
name: variant-record-field-extraction
description: Use when a Nextflow process must read a simulated DNA variant record
  in reference (not ClinVar) format and pull out the gene symbol of the injected pathogenic
  variant for downstream meta or AI scenario conditioning.
when_to_use_negative:
- The variant record is still in ClinVar format and has not been harmonised to the
  reference format — field layout differs and gene extraction would be applied to
  the wrong schema
- You need to detect or call variants from simulated reads (SimuSCoP FASTQ) rather
  than extract annotation fields from an already-simulated variant record
- The task concerns RNA simulation outputs (count matrices, Polyester FASTQ, Salmon
  quantification), which follow a different branch of the pipeline
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0769
tools:
- name: Nextflow
  role: Workflow engine in which the PYCONVERTSIM process module is scaffolded following
    the nf-core/eduomics module structure
  repo: https://github.com/lescai-teaching/eduomics
- name: Docker
  role: Containerisation of the PYCONVERTSIM process so each DSL2 process runs in
    one reproducible container
  repo: https://github.com/lescai-teaching/eduomics
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:fa512d5a3f05f8e7d0958358a2aee75757d90b72b790230947377ea73c058765
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lescai-teaching/eduomics
  tools:
  - Docker
  - Nextflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# variant-record-field-extraction

## Summary

Parse the simulated variant record produced by the eduomics DNA simulation subworkflow to extract the gene name of the injected pathogenic variant, emitting it (e.g. gene_annotation.tsv) so it can populate workflow meta and condition the AI educational scenario prompt.

## When to use

Use when a Nextflow process must read a simulated DNA variant record in reference (not ClinVar) format and pull out the gene symbol of the injected pathogenic variant for downstream meta or AI scenario conditioning.

## When NOT to use

- The variant record is still in ClinVar format and has not been harmonised to the reference format — field layout differs and gene extraction would be applied to the wrong schema
- You need to detect or call variants from simulated reads (SimuSCoP FASTQ) rather than extract annotation fields from an already-simulated variant record
- The task concerns RNA simulation outputs (count matrices, Polyester FASTQ, Salmon quantification), which follow a different branch of the pipeline

## Inputs

- Simulated variant record from the eduomics DNA simulation subworkflow, in reference format (post-SUBVAR harmonisation, not ClinVar format)
- nf-core/eduomics module scaffolding (Nextflow DSL2 process template)
- Docker container definition for the module

## Outputs

- gene_annotation.tsv containing the gene name of the input variant record
- Gene name emitted to the variant-calling workflow meta
- Gene name forwarded as the condition of the AI scenario prompt used for the educational scenario

## How to apply

Scaffold a Nextflow process module (e.g. PYCONVERTSIM) following the nf-core/eduomics module structure, containerised with Docker so one container serves one process. Load the variant record emitted by the DNA simulation subworkflow — after the SUBVAR harmonisation these outputs follow the reference format rather than ClinVar's, so parse reference-format annotation fields rather than ClinVar HGVS naming. Extract the gene name associated with the simulated pathogenic variant and emit it as gene_annotation.tsv. The same gene name is then passed forward as the condition for the AI scenario prompt, which generates the [simulation_id]_scenario.txt file providing biological context for storyline-based learning.

## Related tools

- **Nextflow** (Workflow engine in which the PYCONVERTSIM process module is scaffolded following the nf-core/eduomics module structure) — https://github.com/lescai-teaching/eduomics
- **Docker** (Containerisation of the PYCONVERTSIM process so each DSL2 process runs in one reproducible container) — https://github.com/lescai-teaching/eduomics

## Examples

```
nextflow run eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- gene_annotation.tsv contains exactly the gene name of the input variant record, parsed under the reference format (not ClinVar format)
- The extracted gene matches the gene of the injected pathogenic variant used to teach the variant calling workflow
- The same gene name appears as the condition in the AI scenario prompt and is reflected in the [simulation_id]_scenario.txt output
- The module follows nf-core module structure (one container per process, Docker/Singularity) and passes nf-test/linting as per the eduomics CI
- Downstream variant-calling workflow meta is populated with the extracted gene without schema mismatches

## Limitations

- Variant outputs were harmonised to the reference format rather than ClinVar's; records in the older ClinVar layout will not parse correctly
- The DNA simulation branch focuses on single chromosomes (typically chr22) with capture-based, paired-end sequencing of the human GATK.GRCh38 reference, so gene extraction is scoped to those simulated records
- Only simulations passing the built-in validation (pathogenic variant present in the disease sample reads, absent from the normal sample) are included in final output

## Evidence

- [discussion] SUBVAR module outputs harmonised to reference format instead of ClinVar format: "variant outputs now follow the reference format rather than ClinVar’s. The SUBSET_REFERENCES_TO_TARGET subworkflow was updated accordingly"
- [readme] DNA simulation injects known pathogenic variants whose records the module parses: [evidence span withheld — anchor status: not_found]
- [readme] Gene name conditions the AI educational scenario output: [evidence span withheld — anchor status: not_found]
- [intro] Variant selection originates from ClinVar pathogenic variants within target regions: "Extracts pathogenic variants from ClinVar database within your target regions."
