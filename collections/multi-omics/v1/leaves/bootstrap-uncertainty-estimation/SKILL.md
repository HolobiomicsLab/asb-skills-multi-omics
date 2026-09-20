---
name: bootstrap-uncertainty-estimation
description: Use when a literature-derived biomolecular network model's cell fate
  outputs (e.g. normal proliferation, apoptosis) need a robustness/uncertainty estimate
  under random input perturbations, reported as SEM.
when_to_use_negative:
- The goal is assessing model structure validity rather than numerical robustness
  of fate propensities to input perturbations
- Patient-specific personalized model annotation (drug scores, mutations, RNA-seq
  integration) is required — that is a separate CanSeer workflow step
- Synergy-based drug combination prioritization or therapy response quantification
  is the target output
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: CanSeer
  role: Provides the validated literature-derived biomolecular network model and the
    dynamical analysis (DA) engine run on each of the 256 perturbed bootstrap states
    to obtain cell fate outcomes
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:fac186de34ecc1d6ec40b73eb5b132e7e63818de30bbaa84c059a0c6905716d4
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# bootstrap-uncertainty-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies robustness of cell fate propensity predictions from biomolecular network models by bootstrap-perturbing network inputs (up to 10%), rerunning dynamical analysis, and computing per-fate standard error of the mean (SEM) against a 0.05 SEM threshold.

## When to use

Use when a literature-derived biomolecular network model's cell fate outputs (e.g. normal proliferation, apoptosis) need a robustness/uncertainty estimate under random input perturbations, reported as SEM.

## When NOT to use

- The goal is assessing model structure validity rather than numerical robustness of fate propensities to input perturbations
- Patient-specific personalized model annotation (drug scores, mutations, RNA-seq integration) is required — that is a separate CanSeer workflow step
- Synergy-based drug combination prioritization or therapy response quantification is the target output

## Inputs

- Validated base biomolecular network model from the CanSeer repository
- Base network input values to be perturbed
- Number of bootstrap states (256) and maximum perturbation magnitude (10%)

## Outputs

- Per-cell-fate SEM values (sem_cell_fates.csv)
- Cell fate outcomes from dynamical analysis for each of the 256 perturbed network states
- Robustness verdict per cell fate relative to the 0.05 SEM threshold

## How to apply

Load the validated base network model from the CanSeer repository (github.com/BIRL/CanSeer) and generate 256 bootstrap network states by randomly perturbing the base network input values by up to 10%. Run dynamical analysis (DA) on each of the 256 perturbed states to obtain cell fate outcomes (normal proliferation and apoptosis). For each cell fate, compute the standard error of the mean (SEM) across the 256 states. Judge correctness by comparing computed SEMs to the reported values (normal proliferation SEM 0.0025, apoptosis SEM 0.0021) and verifying they fall below the robustness threshold of SEM 0.05. Write the per-cell-fate SEM values to sem_cell_fates.csv.

## Related tools

- **CanSeer** (Provides the validated literature-derived biomolecular network model and the dynamical analysis (DA) engine run on each of the 256 perturbed bootstrap states to obtain cell fate outcomes) — github.com/BIRL/CanSeer

## Evaluation signals

- Number of bootstrap network states generated equals 256
- Each perturbation of base network input values stays within 10% of the original values
- Computed SEM for normal proliferation approximates 0.0025 and for apoptosis approximates 0.0021
- All per-fate SEM values fall below the 0.05 SEM robustness threshold
- Output file sem_cell_fates.csv contains one SEM entry per cell fate

## Limitations

- SEM-based robustness here is specific to small (up to 10%) random input perturbations of the base network inputs; it does not test structural network validity or patient-data integration quality
- The published model used for demonstration of CanSeer lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes, so perturbation analysis on such patients may be uninformative
- Findings are demonstrated in a lung squamous cell carcinoma (LUSC) context; transfer to other cancer types is not established by this evidence

## Evidence

- [abstract] dynamical analysis workflow: "CanSeer employs patient-specific genetic alterations and RNA-seq data to annotate in silico models followed by dynamical network analyses towards assessment of treatment responses"
- [intro] validated model basis: "The proposed method initiates with the development of literature-derived biomolecular regulatory network models followed by their validation."
- [results] SEM robustness result: [evidence span withheld — anchor status: not_found]
