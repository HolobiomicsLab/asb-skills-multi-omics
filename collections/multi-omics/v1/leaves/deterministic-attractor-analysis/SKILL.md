---
name: deterministic-attractor-analysis
description: Use when a curated biomolecular regulatory network must be iterated to
  its stable attractor states under normal or personalized input conditions to quantify
  cell fate outcome probabilities.
when_to_use_negative:
- The goal is stochastic or ensemble simulation rather than deterministic convergence
  to stable attractor states
- The network has not been validated (no literature-derived model with established
  reference fate outcomes exists), since a published model lacking mutation coverage
  can yield inconclusive outcomes
- Only patient-specific drug effects are of interest without first establishing the
  baseline (normal-condition) attractor distribution
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_2258
tools:
- name: CanSeer
  role: Dynamical network analysis platform that executes the deterministic analysis
    (DA) of the loaded signaling network model and yields cell fate state probabilities
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:0d2049a5aae34e92ff0e9838fa6ab7058c6c298957435c860c82fb6e292b1ee4
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
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# deterministic-attractor-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run deterministic analysis (DA) of a Boolean/weight-based signaling network (e.g., the updated Cho et al. human signaling network with 197 nodes and 744 edges) under defined input conditions and derive cell fate propensities (non-invasive proliferation, apoptotic, metastatic states plus apoptosis) from the converged attractor state distribution.

## When to use

Use when a curated biomolecular regulatory network must be iterated to its stable attractor states under normal or personalized input conditions to quantify cell fate outcome probabilities.

## When NOT to use

- The goal is stochastic or ensemble simulation rather than deterministic convergence to stable attractor states
- The network has not been validated (no literature-derived model with established reference fate outcomes exists), since a published model lacking mutation coverage can yield inconclusive outcomes
- Only patient-specific drug effects are of interest without first establishing the baseline (normal-condition) attractor distribution

## Inputs

- Biomolecular regulatory network model in the simulator's accepted format (e.g., updated Cho et al. human signaling network, 197 nodes / 744 edges)
- Input configuration specifying regulatory node baseline states (normal conditions) or patient-specific genetic alterations and RNA-seq-derived drug score annotations
- Published reference cell fate propensities for validation (e.g., Fig. 2C: NP 0.77, AP 0.08, MT 0.15)

## Outputs

- Converged attractor / stable state probability distribution of the network
- Cell fate propensity values per outcome (non-invasive proliferation, apoptotic, metastatic states, plus apoptosis), e.g., cell_fate_propensities_normal.csv

## How to apply

Load the validated literature-derived network (e.g., the updated Cho et al. human signaling network, 197 nodes and 744 edges) in the simulator's accepted format from the CanSeer repository. Configure regulatory nodes at their unperturbed baseline states for normal conditions — or annotate the model with patient-specific genetic alterations and drug scores (DS) when reproducing personalized scenarios — ensuring no unperturbed patient-specific mutations or drug annotations are applied for the baseline run. Execute the deterministic analysis (DA) mode of the Boolean/weight-based simulator, iterating network dynamics until the attractor/stable state distribution converges. Compute the propensity of each cell fate outcome (non-invasive proliferation, apoptotic, and metastatic states, plus apoptosis) from the stable state probabilities. Write the resulting fate propensities to an output such as cell_fate_propensities_normal.csv and compare them against published reference values (Fig. 2C: NP 0.77, AP 0.08, MT 0.15) within the simulator's convergence tolerance; these propensities then feed downstream computation of treatment efficacy and cytotoxic effect.

## Related tools

- **CanSeer** (Dynamical network analysis platform that executes the deterministic analysis (DA) of the loaded signaling network model and yields cell fate state probabilities) — github.com/BIRL/CanSeer

## Evaluation signals

- Simulated normal-condition propensities match published Fig. 2C reference values (NP 0.77, AP 0.08, MT 0.15) within the simulator's convergence tolerance
- The DA run iterated until the attractor / stable state distribution converged (no residual state flux between iterations)
- Output file cell_fate_propensities_normal.csv contains one propensity per fate and propensities sum to approximately 1 across the three fate outcomes (0.77 + 0.08 + 0.15)
- Loaded network dimensions match the expected model (197 nodes, 744 edges) before analysis is launched
- Downstream treatment efficacy and cytotoxic effect computations consume the same fate propensities, so values must be reproducible across repeated DA runs

## Limitations

- The published model used for demonstration lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes
- Propensity agreement is only guaranteed within the simulator's convergence tolerance, not exactly
- Requires the network in the simulator's accepted format; format mismatches in the curated network prevent the analysis from running

## Evidence

- [other] other: [evidence span withheld — anchor status: not_found]
