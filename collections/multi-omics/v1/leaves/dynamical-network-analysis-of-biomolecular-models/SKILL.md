---
name: dynamical-network-analysis-of-biomolecular-models
description: Use when a validated biomolecular regulatory network model needs to be
  annotated with sample-specific RNA-seq (and/or genetic alteration) data and simulated
  via dynamical analysis to quantify cell fate propensities for treatment-response
  assessment.
when_to_use_negative:
- The input is raw unnormalized RNA-seq data — expressions must be median-normalized
  (or drug-score annotated for treatment scenarios) before model annotation
- The goal is drug repositioning or drug-combination discovery rather than baseline
  cell fate propensity computation from an annotated model
- No validated biomolecular regulatory network model is available; the model must
  first be developed and validated from the literature (CanSeer's first workflow stage)
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: CanSeer
  role: Performs model annotation with patient-specific genomic/transcriptomic data
    and runs the dynamical network analysis (DA) from which cell fate propensities
    are computed
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:49323f2d73e5fc47e2058934cdbe62f95cba8c382b3e4dbde2e3bea8dfb1af18
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
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# dynamical-network-analysis-of-biomolecular-models

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate a validated literature-derived biomolecular regulatory network model with patient/sample-specific genomic and transcriptomic data (e.g. median-normalized RNA-seq expressions) and run CanSeer's dynamical analysis (DA) to simulate network trajectories and compute cell fate propensities (proliferation, apoptosis, quiescence, senescence, cell cycle arrest).

## When to use

Use when a validated biomolecular regulatory network model needs to be annotated with sample-specific RNA-seq (and/or genetic alteration) data and simulated via dynamical analysis to quantify cell fate propensities for treatment-response assessment.

## When NOT to use

- The input is raw unnormalized RNA-seq data — expressions must be median-normalized (or drug-score annotated for treatment scenarios) before model annotation
- The goal is drug repositioning or drug-combination discovery rather than baseline cell fate propensity computation from an annotated model
- No validated biomolecular regulatory network model is available; the model must first be developed and validated from the literature (CanSeer's first workflow stage)

## Inputs

- Validated biomolecular regulatory network model from the CanSeer repository (github.com/BIRL/CanSeer)
- Sample-specific RNA-seq gene expression values, median-normalized (e.g. Case 2 unpaired LUSC normal samples, all 50 normal samples)
- Sample-specific genetic alterations (somatic mutations, CNVs) when available

## Outputs

- Annotated personalized network model
- Dynamical analysis (DA) simulated network trajectories
- Five cell fate propensities per annotated model (proliferation, apoptosis, quiescence, senescence, cell cycle arrest), e.g. written to case2_da_propensities.csv

## How to apply

First load the validated biomolecular regulatory network model from the CanSeer repository (github.com/BIRL/CanSeer). Next load the sample gene expression values — for the Case 2 unpaired LUSC use case, the normal-sample expressions are median-normalized before annotation. Annotate the network model nodes with the median-normalized expressions using CanSeer's model annotation step, then run CanSeer's dynamical analysis (DA) on the annotated model to simulate network trajectories. From the DA outcomes compute the five cell fate propensities (proliferation, apoptosis, quiescence, senescence, cell cycle arrest) and write them to a CSV file (e.g. case2_da_propensities.csv). Validation requires each computed propensity to match the reference value to four decimal places (e.g. proliferation 0.1370, apoptosis 0.2709, quiescence 0.3608, senescence 0.1214, cell cycle arrest 0.0911 for the Case 2 median-annotated normal model).

## Related tools

- **CanSeer** (Performs model annotation with patient-specific genomic/transcriptomic data and runs the dynamical network analysis (DA) from which cell fate propensities are computed) — github.com/BIRL/CanSeer

## Examples

```
python canseer_da.py --model canseer_network --annotation case2_normal_median_normalized_rnaseq.csv --out case2_da_propensities.csv
```

## Evaluation signals

- Each computed cell fate propensity matches the reference value to four decimal places (Case 2 median normal model: proliferation 0.1370, apoptosis 0.2709, quiescence 0.3608, senescence 0.1214, cell cycle arrest 0.0911)
- The five propensities sum to approximately 1.0 (0.1370+0.2709+0.3608+0.1214+0.0911 ≈ 0.9812, within expected rounding/coverage tolerance)
- Output CSV (case2_da_propensities.csv) contains exactly five named fate columns with values in [0, 1]
- The network model used is the validated literature-derived model loaded from the CanSeer repository, and nodes were annotated with median-normalized expressions for all 50 normal samples (Case 2)

## Limitations

- The published model used for demonstration lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes
- Missing CNV/SM/SV information for particular patients is not yet abstracted by the framework
- The framework requires expansion to panomics data to avoid under- or overfitting of a patient's profile

## Evidence

- [results] The dynamical analysis of the Case 2 median-annotated normal model produced cell fate propensities of 0.1370 for proliferation, 0.2709 for apoptosis, 0.3608 for quiescence, 0.1214 for senescence, and 0.0911 for cell cycle arrest.: [evidence span withheld — anchor status: not_found]
