---
name: network-input-perturbation-analysis
description: Use when you need to quantify robustness of a validated biomolecular
  regulatory network model under small input perturbations, using SEM of cell fate
  outcomes against a 0.05 SEM threshold.
when_to_use_negative:
- The goal is annotating patient-specific in silico models with genetic alterations,
  RNA-seq data, and drug scores (DS) rather than testing base-network robustness
- The aim is computing treatment efficacy, cytotoxic effect, or comparing cell fates
  under tailored drug treatment across personalized models
- The input is patient-specific genomic/transcriptomic data (e.g. LUSC use-case samples)
  rather than the validated base network model
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: CanSeer
  role: Provides the literature-derived biomolecular regulatory network model, its
    dynamical analysis (DA) routines, and the cell fate propensity computation used
    for the perturbation-robustness assessment
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:8af08e079f1f918ea25326c303fd3fcdf4b3bdb8c878f7dafad4f272d1a79cfb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BIRL/CanSeer
  tools:
  - CanSeer
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# network-input-perturbation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assesses the robustness of a literature-derived biomolecular signaling network model (Step 1 of CanSeer) by randomly perturbing base network inputs and quantifying the standard error of the mean (SEM) of cell fate propensities across bootstrap network states.

## When to use

Use when you need to quantify robustness of a validated biomolecular regulatory network model under small input perturbations, using SEM of cell fate outcomes against a 0.05 SEM threshold.

## When NOT to use

- The goal is annotating patient-specific in silico models with genetic alterations, RNA-seq data, and drug scores (DS) rather than testing base-network robustness
- The aim is computing treatment efficacy, cytotoxic effect, or comparing cell fates under tailored drug treatment across personalized models
- The input is patient-specific genomic/transcriptomic data (e.g. LUSC use-case samples) rather than the validated base network model

## Inputs

- Validated base biomolecular network model from the CanSeer repository (github.com/BIRL/CanSeer)
- Base network input values to be perturbed (up to 10% random perturbation)
- Bootstrap configuration: 256 perturbed network states

## Outputs

- Cell fate outcomes (normal proliferation, apoptosis) from dynamical analysis of each of the 256 perturbed network states
- Per-cell-fate SEM values written to sem_cell_fates.csv

## How to apply

Load the validated base network model from the CanSeer repository (github.com/BIRL/CanSeer). Generate 256 bootstrap network states by randomly perturbing the base network input values by up to 10%. Run dynamical analysis (DA) on each of the 256 perturbed states to obtain cell fate outcomes (normal proliferation and apoptosis). Compute the standard error of the mean (SEM) across the 256 states for each cell fate. Compare computed SEMs against the reported values (normal proliferation SEM 0.0025, apoptosis SEM 0.0021) and verify they fall below the robustness threshold of SEM 0.05. Write the per-cell-fate SEM values to sem_cell_fates.csv.

## Related tools

- **CanSeer** (Provides the literature-derived biomolecular regulatory network model, its dynamical analysis (DA) routines, and the cell fate propensity computation used for the perturbation-robustness assessment) — github.com/BIRL/CanSeer

## Evaluation signals

- Exactly 256 bootstrap network states are generated, each perturbed by up to 10% of the base input values
- Computed SEM for normal proliferation matches the reported value of 0.0025
- Computed SEM for apoptosis matches the reported value of 0.0021
- All per-cell-fate SEM values fall below the 0.05 SEM robustness threshold
- Output file sem_cell_fates.csv is produced containing the per-cell-fate SEM values

## Limitations

- Robustness is assessed only for the base literature-derived network (Step 1); the published model used for demonstration lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes when personalized
- The workflow targets the two cell fates evaluated (normal proliferation and apoptosis) and may not cover other fate readouts without extension

## Evidence

- [methods] workflow quote from source card: [evidence span withheld — anchor status: not_found]
- [discussion] dynamical analysis coupled with patient data: [evidence span withheld — anchor status: not_found]
