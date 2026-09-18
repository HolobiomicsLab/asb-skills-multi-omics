---
name: gene-expression-based-network-model-annotation
description: Use when a validated biomolecular regulatory network model must be personalized
  with normalized RNA-seq expression values before running dynamical network analyses
  of cell fate or treatment response.
when_to_use_negative:
- Input expression values are raw/unnormalized — annotation requires normalized (e.g.
  median-normalized) expressions for meaningful model behavior
- The goal is drug scoring or therapy-response quantification rather than establishing
  the baseline annotated model — that is a downstream CanSeer step using drug scores
  (DS)
- No validated network model exists for the tissue/cancer type of interest — model
  development and validation must precede annotation
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_0602
tools:
- name: CanSeer
  role: Performs the model annotation step (mapping normalized RNA-seq expressions
    onto network model nodes) and the subsequent dynamical network analysis yielding
    cell fate propensities
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:a796bd652d637ef9521cfe0bba8de2e76213e10db6860229dcccade20b626aa5
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
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Gene-expression-based network model annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate a literature-derived biomolecular regulatory network model with normalized RNA-seq gene expression values (per-sample or cohort-median, e.g. median of 50 normal LUSC samples) so that CanSeer's dynamical analysis can compute cell fate propensities from the personalized model.

## When to use

Use when a validated biomolecular regulatory network model must be personalized with normalized RNA-seq expression values before running dynamical network analyses of cell fate or treatment response.

## When NOT to use

- Input expression values are raw/unnormalized — annotation requires normalized (e.g. median-normalized) expressions for meaningful model behavior
- The goal is drug scoring or therapy-response quantification rather than establishing the baseline annotated model — that is a downstream CanSeer step using drug scores (DS)
- No validated network model exists for the tissue/cancer type of interest — model development and validation must precede annotation

## Inputs

- Validated literature-derived biomolecular regulatory network model from CanSeer repository (github.com/BIRL/CanSeer)
- Median-normalized RNA-seq gene expression values (e.g. median across 50 normal samples, Case 2 unpaired LUSC use case)
- Optionally, patient-specific genetic alterations for integrated annotation

## Outputs

- Expression-annotated network model ready for dynamical analysis
- Cell fate propensities from dynamical analysis (proliferation, apoptosis, quiescence, senescence, cell cycle arrest), e.g. written to case2_da_propensities.csv

## How to apply

Load the validated biomolecular regulatory network model from the CanSeer repository (github.com/BIRL/CanSeer). Prepare gene expression values by median-normalizing them (e.g. median across all 50 normal samples in the unpaired Case 2 LUSC use case, or CanSeer's patient-specific normalized expressions). Use CanSeer's model annotation step to map each normalized gene expression onto the corresponding network model node, integrating patient-specific genetic alterations and RNA-seq data. Then run CanSeer's dynamical analysis (DA) on the annotated model to simulate network trajectories and compute the five cell fate propensities (proliferation, apoptosis, quiescence, senescence, cell cycle arrest). The rationale is that annotation with omics data converts a generic literature-derived model into a patient- or cohort-specific model whose dynamical behavior reflects the measured molecular profile.

## Related tools

- **CanSeer** (Performs the model annotation step (mapping normalized RNA-seq expressions onto network model nodes) and the subsequent dynamical network analysis yielding cell fate propensities) — github.com/BIRL/CanSeer

## Examples

```
python run_canseer_da.py --model canseer_network --annotation case2_normal_median_normalized_expression.csv --out case2_da_propensities.csv
```

## Evaluation signals

- Each computed cell fate propensity matches the reported value to four decimal places (Case 2 normal: proliferation 0.1370, apoptosis 0.2709, quiescence 0.3608, senescence 0.1214, cell cycle arrest 0.0911)
- The five propensities sum to approximately 1.0 (0.1370 + 0.2709 + 0.3608 + 0.1214 + 0.0911 = 1.0012, consistent with rounding to 4 dp)
- Annotated model outputs are written in the expected schema (case2_da_propensities.csv with the five fate columns)
- Every gene expression value used for annotation maps to a node present in the network model; nodes lacking coverage should be flagged (inconclusive outcomes can result from lacking model coverage)

## Limitations

- The published model used for demonstration lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes
- The framework requires expansion to cater for panomics data to avoid under- or overfitting of the patient's profile
- Normalization choice (e.g. median across all 50 normal samples vs patient-specific) changes the annotated model's dynamical behavior; the normalization scheme must match the intended use case

## Evidence
