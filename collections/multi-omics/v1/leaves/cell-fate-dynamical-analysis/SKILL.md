---
name: cell-fate-dynamical-analysis
description: Use when a validated biomolecular network model must be simulated via
  dynamical analysis to obtain cell fate outcomes (normal proliferation, apoptosis),
  or when its robustness to small input perturbations must be quantified by SEM.
when_to_use_negative:
- The input is raw RNA-seq or mutation data with no validated biomolecular network
  model available to simulate
- The goal is computing patient-specific drug scores (DS), repositioning FDA-approved
  drugs, or predicting drug combinations rather than quantifying cell fate propensities
  and their robustness
- The published model lacks coverage for the patient's mutations of interest, which
  the article notes resulted in inconclusive outcomes
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_2640
tools:
- name: CanSeer
  role: Provides the validated biomolecular network model and runs the dynamical analysis
    (DA) on each of the 256 perturbed network states to obtain cell fate outcomes
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:9a6d81d2acde97339901e15cf5619fd4d4f2b59f2161ea96722a2f74ab3296ae
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# cell-fate-dynamical-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs dynamical analysis (DA) on the literature-derived biomolecular network model — optionally perturbed into 256 bootstrap states with up to 10% random input variation — to quantify cell fate propensities (normal proliferation, apoptosis) and verify robustness via SEM below a 0.05 threshold.

## When to use

Use when a validated biomolecular network model must be simulated via dynamical analysis to obtain cell fate outcomes (normal proliferation, apoptosis), or when its robustness to small input perturbations must be quantified by SEM.

## When NOT to use

- The input is raw RNA-seq or mutation data with no validated biomolecular network model available to simulate
- The goal is computing patient-specific drug scores (DS), repositioning FDA-approved drugs, or predicting drug combinations rather than quantifying cell fate propensities and their robustness
- The published model lacks coverage for the patient's mutations of interest, which the article notes resulted in inconclusive outcomes

## Inputs

- Validated literature-derived biomolecular network model from the CanSeer repository (github.com/BIRL/CanSeer)
- Patient-specific genetic alterations and RNA-seq data for personalized model annotation (optional)
- Perturbation settings: number of bootstrap network states (256) and maximum random input perturbation (10%)

## Outputs

- Cell fate propensities (normal proliferation and apoptosis) for each of the 256 perturbed network states
- Per-cell-fate SEM values written to sem_cell_fates.csv, compared against the 0.05 SEM robustness threshold

## How to apply

Load the validated base network model from the CanSeer repository (github.com/BIRL/CanSeer); if modeling a patient, first annotate it with patient-specific genetic alterations and RNA-seq data. Perturb the base network input values randomly by up to 10% to generate each of 256 bootstrap network states, so that stability of cell fate predictions — not a single point estimate — is tested. Run dynamical analysis (DA) on each of the 256 perturbed states to obtain cell fate outcomes for normal proliferation and apoptosis. Compute the standard error of the mean (SEM) across the 256 states for each cell fate and compare against the reported values (normal proliferation SEM 0.0025, apoptosis SEM 0.0021) and the robustness threshold of SEM 0.05. Write the per-cell-fate SEM values to sem_cell_fates.csv; SEMs above 0.05 indicate the network model is not robust and downstream computations of treatment efficacy and cytotoxic effect from cell fate outcomes should be paused pending review.

## Related tools

- **CanSeer** (Provides the validated biomolecular network model and runs the dynamical analysis (DA) on each of the 256 perturbed network states to obtain cell fate outcomes) — github.com/BIRL/CanSeer

## Evaluation signals

- Exactly 256 bootstrap network states are generated, each with input values perturbed by up to 10%
- Both cell-fate SEMs (normal proliferation, apoptosis) fall below the 0.05 SEM robustness threshold
- Computed SEMs reproduce the reported values: normal proliferation SEM 0.0025 and apoptosis SEM 0.0021
- A sem_cell_fates.csv file is produced with one SEM value per cell fate
- DA completes on every perturbed state and yields a cell fate outcome for each state

## Limitations

- Robustness is defined against random input perturbations up to 10% across 256 bootstrap states; other perturbation magnitudes or state counts will change the SEM values
- The published model used for demonstration lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes
- Robustness was only quantified for the two cell fates assessed (normal proliferation and apoptosis); other fate readouts are not covered by the reported SEM values

## Evidence

- [other] Bootstrap states are generated by perturbing base network inputs by up to 10%: [evidence span withheld — anchor status: not_found]
- [other] SEM robustness: normal proliferation 0.0025, apoptosis 0.0021, both below the 0.05 threshold: [evidence span withheld — anchor status: not_found]
- [discussion] Dynamical analysis couples the network model with patient-specific genomic and transcriptomic data: [evidence span withheld — anchor status: not_found]
- [intro] Cell fate outcomes feed into treatment efficacy and cytotoxic effect computation: [evidence span withheld — anchor status: not_found]
