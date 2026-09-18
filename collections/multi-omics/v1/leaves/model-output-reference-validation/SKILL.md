---
name: model-output-reference-validation
description: Use when a literature-derived signaling network (e.g., the updated Cho
  et al. human signaling network) must be re-analyzed to reproduce originally published
  cell fate propensities before applying patient-specific annotations or drug perturbations.
when_to_use_negative:
- 'The goal is personalized analysis: once patient-specific genetic alterations, RNA-seq
  annotations, or drug scores are applied, outcomes are no longer comparable to the
  published normal-condition baseline.'
- The network is not literature-derived or lacks published reference propensities,
  so no reference values exist to validate against.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: CanSeer
  role: Executes the deterministic analysis (DA) mode of the Boolean/weight-based
    simulator on the literature-derived network under normal input conditions and
    reports cell fate propensities for baseline validation.
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:66c0b7078a67c42c21344c802685b9a777702162b0190a15e7d97782cd723992
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BIRL/CanSeer
  repo_url: https://github.com/BIRL/CanSeer
  tools:
  - CanSeer
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# model-output-reference-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates a biomolecular network model by running its deterministic analysis (DA) under unperturbed normal input conditions and comparing the resulting cell fate propensities against published reference values, using CanSeer.

## When to use

Use when a literature-derived signaling network (e.g., the updated Cho et al. human signaling network) must be re-analyzed to reproduce originally published cell fate propensities before applying patient-specific annotations or drug perturbations.

## When NOT to use

- The goal is personalized analysis: once patient-specific genetic alterations, RNA-seq annotations, or drug scores are applied, outcomes are no longer comparable to the published normal-condition baseline.
- The network is not literature-derived or lacks published reference propensities, so no reference values exist to validate against.

## Inputs

- Updated Cho et al. human signaling network model (197 nodes, 744 edges) from the CanSeer repository in the simulator's accepted format
- Normal input condition configuration (all regulatory nodes at unperturbed baseline states, no mutations or drug annotations)
- Reference cell fate propensity values (NP 0.77, AP 0.08, MT 0.15; Fig. 2C)

## Outputs

- cell_fate_propensities_normal.csv containing fate propensities for non-invasive proliferation, apoptosis, and metastasis
- Converged attractor/cell fate state distribution from the DA run

## How to apply

Load the updated Cho et al. human signaling network (197 nodes, 744 edges) from the CanSeer repository in the simulator's accepted format. Leave all regulatory nodes at their unperturbed baseline states — no patient-specific mutations, CNV/RNA-seq annotations, or drug scores — then execute deterministic analysis (DA) in CanSeer, iterating the network dynamics until the attractor/cell fate state distribution converges. Compute the propensity of each cell fate (non-invasive proliferation, apoptotic, and metastatic states, plus apoptosis) from the simulated stable state probabilities and write them to cell_fate_propensities_normal.csv. Judge success by whether the simulated propensities match the published normal-condition reference values (NP 0.77, AP 0.08, MT 0.15 from Fig. 2C) within the simulator's convergence tolerance. This baseline reproduction is the validation step CanSeer prescribes before integrating patient-specific genomic and transcriptomic data for personalized dynamical analysis.

## Related tools

- **CanSeer** (Executes the deterministic analysis (DA) mode of the Boolean/weight-based simulator on the literature-derived network under normal input conditions and reports cell fate propensities for baseline validation.) — github.com/BIRL/CanSeer

## Evaluation signals

- Simulated propensity for normal proliferation equals 0.77 within the simulator's convergence tolerance
- Simulated propensity for abnormal proliferation (apoptotic state) equals 0.08 within tolerance
- Simulated propensity for metastasis equals 0.15 within tolerance
- DA run reached a converged attractor/cell fate state distribution rather than stopping early
- Input configuration confirms all regulatory nodes were left at unperturbed baseline states (no mutations, CNV/RNA-seq, or drug-score annotations applied)

## Limitations

- The published model used for demonstration lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes — model coverage must be checked before extending beyond the normal-condition baseline.
- In silico cell fate predictions are computational only and require further evaluation (e.g., wet lab or tumor-on-chip models) before clinical interpretation.
- Validation is only meaningful against the specific reference values (Fig. 2C: NP 0.77, AP 0.08, MT 0.15) reported for normal input conditions of this particular network.

## Evidence

- [discussion] CanSeer couples dynamical analysis of biomolecular network models: [evidence span withheld — anchor status: not_found]
