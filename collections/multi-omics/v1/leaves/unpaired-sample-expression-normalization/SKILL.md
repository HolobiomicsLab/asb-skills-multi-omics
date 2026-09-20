---
name: unpaired-sample-expression-normalization
description: Use when normal and cancer samples come from different patients (unpaired
  design, CanSeer Case 2) and normal-sample RNA-seq expressions must be summarized
  (median-normalized) before annotating an in silico network model.
when_to_use_negative:
- The use case is paired samples (Case 1), where patient-matched normal samples are
  available and per-patient pairing rather than median over all normals applies
- The use case is cancer samples only (Case 3), which provides no normal samples to
  compute a median-normalized reference from
- The expression data is already median-normalized / has been previously annotated
  and re-normalizing would alter the annotated node values
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: CanSeer
  role: Provides the model annotation step that accepts the median-normalized normal-sample
    expressions and annotates the validated network model nodes before dynamical network
    analysis
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:5889e3002971c0bde080c5860692d1e7380b91667b40bc3e19423b09bfd6f83d
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
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Unpaired-sample expression normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Median-normalizes RNA-seq gene expression values of all normal samples in the unpaired (Case 2) LUSC use case so the resulting values can annotate nodes of the validated CanSeer biomolecular regulatory network model prior to dynamical analysis.

## When to use

Use when normal and cancer samples come from different patients (unpaired design, CanSeer Case 2) and normal-sample RNA-seq expressions must be summarized (median-normalized) before annotating an in silico network model.

## When NOT to use

- The use case is paired samples (Case 1), where patient-matched normal samples are available and per-patient pairing rather than median over all normals applies
- The use case is cancer samples only (Case 3), which provides no normal samples to compute a median-normalized reference from
- The expression data is already median-normalized / has been previously annotated and re-normalizing would alter the annotated node values

## Inputs

- RNA-seq gene expression values (raw or normalized) for the 50 normal samples of the Case 2 unpaired-samples LUSC use case
- Validated biomolecular regulatory network model from github.com/BIRL/CanSeer
- Case label / use-case specification (Case 2: unpaired samples)

## Outputs

- Median-normalized gene expression vector (one value per gene across all normal samples) used for model node annotation
- Annotated CanSeer network model ready for dynamical analysis
- case2_da_propensities.csv containing the five cell fate propensities

## How to apply

Load the literature-derived, validated biomolecular regulatory network model from the CanSeer repository (github.com/BIRL/CanSeer). Load the RNA-seq gene expression values of all 50 normal samples belonging to the Case 2 unpaired-samples use case, where normal and tumor samples are drawn from different patients. Compute the median of each gene's expression across the normal samples to obtain a single median-normalized expression vector per gene. Annotate the network model nodes with these median-normalized normal-sample expressions using CanSeer's model annotation step, then run CanSeer's dynamical analysis (DA) to simulate network trajectories. From the DA outcomes, compute the five cell fate propensities (proliferation, apoptosis, quiescence, senescence, cell cycle arrest) and write them to case2_da_propensities.csv. Rationale: in the unpaired setting no matched normal exists per tumor, so median normalization across all normal samples provides the population-level reference signature used to contextualize the cancer models.

## Related tools

- **CanSeer** (Provides the model annotation step that accepts the median-normalized normal-sample expressions and annotates the validated network model nodes before dynamical network analysis) — github.com/BIRL/CanSeer

## Evaluation signals

- Each of the five computed cell fate propensities matches the reported Case 2 median normal model values to four decimal places: proliferation 0.1370, apoptosis 0.2709, quiescence 0.3608, senescence 0.1214, cell cycle arrest 0.0911
- The five cell fate propensities are complete (all five fates reported) and sum to approximately 1.0
- The normalization uses all 50 normal samples of the Case 2 unpaired use case (sample count check), not a subset
- Output file case2_da_propensities.csv exists and contains one row/column per cell fate
- The annotated model nodes correspond one-to-one to the genes present in the median-normalized expression input

## Limitations

- The published model used for demonstration of CanSeer lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes
- The framework requires an expansion to cater for panomics data, to avoid under- or overfitting of the patient's profile, so median-normalized RNA-seq alone may incompletely represent the molecular signature
- Normalization behavior is specific to the unpaired (Case 2) design; applying it to paired or cancer-only use cases is out of scope

## Evidence

- [other] Load the Case 2 (unpaired-samples use case) normal-sample gene expression values median-normalized for annotation.: [evidence span withheld — anchor status: not_found]
- [other] verify each computed propensity matches the reported value (proliferation 0.1370, apoptosis 0.2709, quiescence 0.3608, senescence 0.1214, cell cycle arrest 0.0911): [evidence span withheld — anchor status: not_found]
