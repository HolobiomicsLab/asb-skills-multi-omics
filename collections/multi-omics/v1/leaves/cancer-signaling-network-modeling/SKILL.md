---
name: cancer-signaling-network-modeling
description: Use when reproducing or validating cell fate propensities (normal proliferation,
  abnormal proliferation/apoptotic, metastatic states) from a literature-derived cancer
  signaling network via deterministic analysis under baseline or patient-annotated
  input conditions.
when_to_use_negative:
- When the aim is patient-specific drug scoring and therapy-response prediction rather
  than baseline network reproduction — that requires the CanSeer drug score (DS) annotation
  workflow on top of the network model
- When the network model lacks coverage for the mutations or nodes of interest; the
  article notes the published model used for demonstration lacked coverage for certain
  mutations in some patients, resulting in inconclusive outcomes
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: CanSeer
  role: Simulator used to load the literature-derived biomolecular regulatory network,
    run dynamical (deterministic) network analyses, and compute cell fate outcomes
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:7c1a9df33975792a0cee40a11693f0ead670c45b1d9f67c014287ff2190b9dc4
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
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cancer-signaling-network-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Build and simulate literature-derived cancer biomolecular regulatory network models (e.g., the updated Cho et al. human signaling network with 197 nodes and 744 edges) using CanSeer's dynamical network analyses to quantify cell fate propensities (proliferation, apoptosis, metastasis) under defined input conditions.

## When to use

Use when reproducing or validating cell fate propensities (normal proliferation, abnormal proliferation/apoptotic, metastatic states) from a literature-derived cancer signaling network via deterministic analysis under baseline or patient-annotated input conditions.

## When NOT to use

- When the aim is patient-specific drug scoring and therapy-response prediction rather than baseline network reproduction — that requires the CanSeer drug score (DS) annotation workflow on top of the network model
- When the network model lacks coverage for the mutations or nodes of interest; the article notes the published model used for demonstration lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes

## Inputs

- Cancer signaling network model file in the simulator's accepted format (e.g., updated Cho et al. human signaling network, 197 nodes / 744 edges, from the CanSeer repository)
- Input-condition configuration (normal/unperturbed baseline, or patient-specific genetic alterations and RNA-seq annotations)
- Published reference cell fate propensities for validation (e.g., Fig. 2C: NP 0.77, AP 0.08, MT 0.15)

## Outputs

- cell_fate_propensities_normal.csv containing propensity values for each cell fate (non-invasive proliferation, apoptotic, metastatic states, apoptosis)
- Converged attractor/cell fate state distribution from the deterministic analysis

## How to apply

Load the literature-derived biomolecular regulatory network model (e.g., the updated Cho et al. human signaling network, 197 nodes / 744 edges) from the CanSeer repository in the simulator's accepted format. Configure inputs — for baseline reproduction, leave all regulatory nodes unperturbed with no patient-specific mutations or drug annotations; for personalization, annotate the model with patient-specific genetic alterations and RNA-seq data. Execute the deterministic analysis (DA) mode, iterating the Boolean/weight-based dynamics until the attractor/cell fate state distribution converges. Compute the propensity of each cell fate (non-invasive proliferation, apoptotic, and metastatic states, plus apoptosis) from the stable state probabilities and write them to cell_fate_propensities_normal.csv. Validate against published reference values (NP 0.77, AP 0.08, MT 0.15 from Fig. 2C) within the simulator's convergence tolerance.

## Related tools

- **CanSeer** (Simulator used to load the literature-derived biomolecular regulatory network, run dynamical (deterministic) network analyses, and compute cell fate outcomes) — github.com/BIRL/CanSeer

## Evaluation signals

- Simulated propensities under normal input conditions match published values NP 0.77, AP 0.08, MT 0.15 (Fig. 2C) within the simulator's convergence tolerance
- The attractor/cell fate state distribution from the DA run has converged before propensities are computed
- Cell fate propensity values are written to cell_fate_propensities_normal.csv in the expected fate categories (proliferation, apoptotic, metastatic, plus apoptosis)
- Propensities sum to 1 across the three main fates (0.77 + 0.08 + 0.15) in the baseline reproduction

## Limitations

- The published model used for demonstration of CanSeer lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes
- In silico cell fate predictions require further evaluation (the article notes predicted patient-specific drug responses need tumor-on-chip validation); no wet lab validation is yet reported
- A multi-factorial patient data-integrative approach to simulate drug effects followed by quantification of therapy response had not previously been assessed, so reference baselines may be model-specific

## Evidence

- [abstract] CanSeer employs patient-specific genetic alterations and RNA-seq data to annotate in silico models followed by dynamical network analyses towards assessment of treatment responses: "CanSeer employs patient-specific genetic alterations and RNA-seq data to annotate in silico models followed by dynamical network analyses towards assessment of treatment responses"
- [intro] The proposed method initiates with the development of literature-derived biomolecular regulatory network models followed by their validation.: "The proposed method initiates with the development of literature-derived biomolecular regulatory network models followed by their validation."
- [other] deterministic analysis identified three cell fates with propensities of 0.77 for normal proliferation, 0.08 for abnormal proliferation, and 0.15 for metastasis: [evidence span withheld — anchor status: not_found]
- [discussion] the published model used for demonstration of CanSeer lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes: "the published model used for demonstration of CanSeer lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes"
