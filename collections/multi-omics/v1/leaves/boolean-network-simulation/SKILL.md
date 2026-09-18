---
name: boolean-network-simulation
description: Use when a validated, literature-derived regulatory network must be iterated
  to a stable state distribution to quantify cell fate propensities under defined
  input conditions (e.g., normal baseline, patient mutations, or drug annotations).
when_to_use_negative:
- The published model lacks coverage for certain patient mutations, which can result
  in inconclusive outcomes — validate node coverage before annotating patient-specific
  alterations
- The goal is synergy-based drug combination prioritization, which CanSeer's analysis
  remains limited in supporting
- Only a static interaction map is available without a validated, executable regulatory
  model — the network must first be developed and validated before dynamical analysis
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CanSeer
  role: Executes the deterministic analysis (DA) mode of the Boolean/weight-based
    simulator on the annotated network model and computes cell fate propensities
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:d99bdfea195c112e6290a456c84b20347ca95fcc8a106c8db554770febfbd7c8
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# boolean-network-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulates a literature-derived biomolecular regulatory network (e.g., the Cho et al. human signaling network of 197 nodes and 744 edges) as a Boolean/weight-based model using CanSeer's deterministic analysis (DA) mode, converging on attractor states to quantify cell fate propensities (normal proliferation, apoptosis, metastasis).

## When to use

Use when a validated, literature-derived regulatory network must be iterated to a stable state distribution to quantify cell fate propensities under defined input conditions (e.g., normal baseline, patient mutations, or drug annotations).

## When NOT to use

- The published model lacks coverage for certain patient mutations, which can result in inconclusive outcomes — validate node coverage before annotating patient-specific alterations
- The goal is synergy-based drug combination prioritization, which CanSeer's analysis remains limited in supporting
- Only a static interaction map is available without a validated, executable regulatory model — the network must first be developed and validated before dynamical analysis

## Inputs

- Biomolecular regulatory network model in CanSeer's accepted format (e.g., updated Cho et al. human signaling network, 197 nodes / 744 edges, from the CanSeer repository)
- Configuration of input/regulatory node states (normal/unperturbed baseline, or patient-specific mutations and drug annotations)
- Optional: patient-specific genetic alterations and RNA-seq data for personalized model annotation
- Optional: drug scores (DS) computed from drug activity scores and normalized gene expressions

## Outputs

- cell_fate_propensities_normal.csv with propensity values per cell fate (e.g., 0.77 normal proliferation, 0.08 apoptosis, 0.15 metastasis under normal conditions)
- Converged attractor/stable state distribution of the network
- Cell fate outcomes used downstream to compute treatment efficacy and cytotoxic effect

## How to apply

Load the curated network (e.g., the updated Cho et al. human signaling network with 197 nodes and 744 edges) into CanSeer in the simulator's accepted format, leaving all regulatory nodes at unperturbed baseline states for normal-condition runs. Execute the deterministic analysis (DA) mode of the Boolean/weight-based simulator, iterating the network dynamics until the attractor/cell fate state distribution converges. From the simulated stable state probabilities, compute the propensity of each cell fate outcome (non-invasive proliferation, apoptotic, and metastatic states, plus apoptosis) and write them to cell_fate_propensities_normal.csv. For personalized runs, annotate the same network with patient-specific genetic alterations (CNV, somatic mutations) and RNA-seq data, and/or drug scores (DS) computed from drug activity scores and normalized gene expressions, then re-analyze. The rationale is that dynamical analysis of the validated network is CanSeer's core engine for assessing treatment efficacy, cytotoxic effect, and patient-to-patient therapeutic response variability.

## Related tools

- **CanSeer** (Executes the deterministic analysis (DA) mode of the Boolean/weight-based simulator on the annotated network model and computes cell fate propensities) — github.com/BIRL/CanSeer

## Evaluation signals

- Under normal input conditions, simulated propensities must match Cho et al.'s published Fig. 2C values (NP 0.77, AP 0.08, MT 0.15) within the simulator's convergence tolerance
- The network loaded must contain the expected 197 nodes and 744 edges
- The DA simulation must iterate to a converged attractor/cell fate state distribution before propensities are computed
- Output file cell_fate_propensities_normal.csv must contain one propensity per modeled cell fate (non-invasive proliferation, apoptotic, metastatic, plus apoptosis), each between 0 and 1
- Propensities should sum (approximately) to 1 across the modeled fate states

## Limitations

- The published model used for demonstration lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes
- In silico predicted patient-specific drug sensitivity requires further evaluation with tumor-on-chip models; no wet lab validation of predicted efficacious combinations yet
- The study remains limited in prioritizing synergy-based drug combinations

## Evidence

- [abstract] CanSeer employs patient-specific genetic alterations and RNA-seq data to annotate in silico models: "CanSeer employs patient-specific genetic alterations and RNA-seq data to annotate in silico models"
- [results] DA reproduction of Cho et al. cell fate propensities: [evidence span withheld — anchor status: not_found]
- [intro] Network model development precedes dynamical analysis: "The proposed method initiates with the development of literature-derived biomolecular regulatory network models followed by their validation."
- [intro] Cell fate outcomes feed treatment efficacy computation: "Employing network model’s cell fate outcomes, treatment efficacy and cytotoxic effect are computed, which then elucidates treatment suitability."
- [discussion] Dynamical network analysis for treatment response: "CanSeer couples’ dynamical analysis of biomolecular network models with patient-specific genomic and transcriptomic data to assess the individualized therapeutic responses"
