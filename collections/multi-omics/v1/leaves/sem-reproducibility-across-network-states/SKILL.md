---
name: sem-reproducibility-across-network-states
description: 'Use when validating a dynamical biomolecular network model''s stability:
  you need to show cell fate predictions (e.g., proliferation, apoptosis) are insensitive
  to small (up to 10%) input perturbations via SEM.'
when_to_use_negative:
- The model has not yet been developed and validated (Step 1 of CanSeer is a prerequisite;
  the base network must already exist)
- The goal is to annotate the model with patient-specific genomic/transcriptomic data
  or compute drug scores (DS) for treatment response, rather than to assess baseline
  model robustness to input perturbations
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_2259
tools:
- name: CanSeer
  role: Provides the validated literature-derived biomolecular network model and the
    dynamical analysis (DA) procedure run on each perturbed network state to obtain
    cell fate outcomes
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:14102beae70d52ffac71b62badfe026841516e787a41f652faea9159848ba842
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
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# sem-reproducibility-across-network-states

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the robustness of a literature-derived biomolecular signaling network model (Step 1 of CanSeer) by perturbing its inputs to generate bootstrap network states and computing the standard error of the mean (SEM) of cell fate propensities (normal proliferation and apoptosis) across those states.

## When to use

Use when validating a dynamical biomolecular network model's stability: you need to show cell fate predictions (e.g., proliferation, apoptosis) are insensitive to small (up to 10%) input perturbations via SEM.

## When NOT to use

- The model has not yet been developed and validated (Step 1 of CanSeer is a prerequisite; the base network must already exist)
- The goal is to annotate the model with patient-specific genomic/transcriptomic data or compute drug scores (DS) for treatment response, rather than to assess baseline model robustness to input perturbations

## Inputs

- Validated base literature-derived biomolecular regulatory network model from the CanSeer repository (github.com/BIRL/CanSeer)
- Perturbation parameters: maximum 10% random input perturbation and number of bootstrap states (256)

## Outputs

- Per-cell-fate SEM values (normal proliferation and apoptosis) written to sem_cell_fates.csv
- Robustness verdict per cell fate: SEM below the 0.05 threshold

## How to apply

Load the validated base network model from the CanSeer repository (github.com/BIRL/CanSeer) and randomly perturb its input values by up to 10% to generate 256 bootstrap network states. Run dynamical analysis (DA) on each of the 256 perturbed states to obtain cell fate outcomes (normal proliferation and apoptosis). Compute the standard error of the mean (SEM) across the 256 states for each cell fate. Judge correctness by comparing computed SEMs against the reported values (normal proliferation SEM 0.0025, apoptosis SEM 0.0021) and verifying both fall below the robustness threshold of SEM 0.05. Write the per-cell-fate SEM values to sem_cell_fates.csv.

## Related tools

- **CanSeer** (Provides the validated literature-derived biomolecular network model and the dynamical analysis (DA) procedure run on each perturbed network state to obtain cell fate outcomes) — github.com/BIRL/CanSeer

## Evaluation signals

- Computed SEM for normal proliferation matches the reported value of 0.0025 (within expected run-to-run variation)
- Computed SEM for apoptosis matches the reported value of 0.0021 (within expected run-to-run variation)
- All per-cell-fate SEMs fall below the robustness threshold of SEM 0.05
- sem_cell_fates.csv contains SEM entries for each cell fate (normal proliferation and apoptosis) across the 256 bootstrap states
- Exactly 256 perturbed network states are generated, each with inputs perturbed by no more than 10%

## Limitations

- Robustness here is measured only as SEM of cell fate propensities under up-to-10% input perturbations; it does not capture structural uncertainty in the network topology
- The skill reflects baseline model stability (Step 1); it does not assess robustness of patient-specific annotated models or drug response predictions
- Results are bootstrap-based (256 states), so SEM estimates carry sampling variability and small run-to-run differences from the reported values (0.0025, 0.0021) are expected

## Evidence

- [results] The network model is robust to input perturbations: the highest variation across cell fates was in normal proliferation (SEM 0.0025), followed by apoptosis (SEM 0.0021), both far below a 0.05 SEM robustness threshold.: [evidence span withheld — anchor status: not_found]
