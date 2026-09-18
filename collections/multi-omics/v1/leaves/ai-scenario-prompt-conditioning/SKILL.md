---
name: ai-scenario-prompt-conditioning
description: Use when a simulated DNA variant record from the eduomics pipeline must
  yield a gene name that conditions the AI educational scenario for storyline-based
  learning of variant calling workflows.
when_to_use_negative:
- The variant record is still in ClinVar format (pre-harmonisation SUBVAR output)
  — convert/harmonise to reference format first, as eduomics variant outputs now follow
  the reference format rather than ClinVar's
- The goal is variant-calling validation itself (checking the injected pathogenic
  variant is present in disease reads and absent from normal reads) rather than generating
  educational scenario context
- Working in the RNA simulation branch, where scenario conditioning derives from simulated
  differential expression patterns rather than a single pathogenic variant gene
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Nextflow
  role: Workflow engine in which the PYCONVERTSIM process module is scaffolded and
    its gene-name output channeled to the AI scenario prompt and workflow meta
  repo: https://www.nextflow.io/
- name: Docker
  role: Container system used to run the containerised PYCONVERTSIM module reproducibly
  repo: https://www.docker.com/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:4517f178c49c5eb2c5c2b2c78487626d114f46a6375ad6cd639bc3ed9609eb30
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - Nextflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.0
n_claims: 6
---

# ai-scenario-prompt-conditioning

## Summary

Extracts the gene name from an eduomics simulated DNA variant record (post-harmonisation reference format) and passes it forward as the biological condition for the AI-generated educational scenario used in storyline-based learning.

## When to use

Use when a simulated DNA variant record from the eduomics pipeline must yield a gene name that conditions the AI educational scenario for storyline-based learning of variant calling workflows.

## When NOT to use

- The variant record is still in ClinVar format (pre-harmonisation SUBVAR output) — convert/harmonise to reference format first, as eduomics variant outputs now follow the reference format rather than ClinVar's
- The goal is variant-calling validation itself (checking the injected pathogenic variant is present in disease reads and absent from normal reads) rather than generating educational scenario context
- Working in the RNA simulation branch, where scenario conditioning derives from simulated differential expression patterns rather than a single pathogenic variant gene

## Inputs

- Simulated variant record from the eduomics DNA simulation subworkflow (reference-format variant output, not ClinVar format)
- nf-core/eduomics Nextflow module structure and meta map (workflow meta)

## Outputs

- gene_annotation.tsv containing the gene name of the input variant record
- Gene name propagated into the variant-calling workflow meta
- [simulation_id]_scenario.txt — AI-generated educational scenario conditioned on the extracted gene name

## How to apply

Scaffold a Nextflow process module named PYCONVERTSIM following the nf-core/eduomics module structure and containerise it with Docker. Load the variant record emitted by the DNA simulation subworkflow — per the paper's harmonisation (SUBVAR), this record follows the reference format rather than ClinVar's format. Parse the record fields to extract the gene name of the simulated pathogenic variant, then emit it as gene_annotation.tsv so it can populate the variant-calling workflow meta. Finally, pass the same gene name as the condition for the AI scenario prompt, which writes a [simulation_id]_scenario.txt file providing biological context for the simulation. The rationale is that the 'ground truth' gene is the one injected as a known pathogenic variant, so the scenario and the variant-calling exercise must share it.

## Related tools

- **Nextflow** (Workflow engine in which the PYCONVERTSIM process module is scaffolded and its gene-name output channeled to the AI scenario prompt and workflow meta) — https://www.nextflow.io/
- **Docker** (Container system used to run the containerised PYCONVERTSIM module reproducibly) — https://www.docker.com/

## Examples

```
nextflow run nf-core/eduomics -profile docker --input dna_samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- gene_annotation.tsv contains exactly the gene name of the input variant record in reference format
- The extracted gene name matches the gene of the injected pathogenic variant used to teach the variant calling workflow
- A [simulation_id]_scenario.txt file is produced for each DNA simulation and its biological context is conditioned on the same gene name
- The gene name correctly populates the variant-calling workflow meta without format drift from ClinVar to reference format
- Only simulations passing validation (injected variant detected in disease sample, absent from normal sample) feed the scenario generation

## Limitations

- Eduomics focuses on single chromosomes (typically chr22) with capture-based, paired-end sequencing against a human reference (GATK.GRCh38 recommended), so scenario conditioning is limited to genes within the simulated target regions
- Variant outputs were harmonised from ClinVar format to the reference format (SUBVAR module update), so older ClinVar-format records cannot be used directly
- Scenarios are AI-generated educational contexts; their biological content depends on the prompt condition and is not an independently validated ground truth

## Evidence

- [full_text] Each DNA simulation outputs a [simulation_id]_scenario.txt file, which is an AI-generated educational scenario providing biological context for the simulation.: [evidence span withheld — anchor status: not_found]
- [discussion] variant outputs now follow the reference format rather than ClinVar's: "variant outputs now follow the reference format rather than ClinVar's"
- [readme] AI-Powered Scenarios: Automatically generate educational contexts and case studies: [evidence span withheld — anchor status: not_found]
- [intro] Checks that the simulated pathogenic variant is indeed present in the reads of the "disease" sample and absent from the "normal" sample: "Checks that the simulated pathogenic variant is indeed present in the reads of the "disease" sample and absent from the "normal" sample"
