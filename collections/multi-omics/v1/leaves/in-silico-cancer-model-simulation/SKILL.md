---
name: in-silico-cancer-model-simulation
description: Use when a validated biomolecular regulatory network model (e.g.
when_to_use_negative:
- You need wet-lab or clinical validation of predicted efficacious drug combinations
  — the article notes such validation is not yet available for CanSeer predictions
- The published network model lacks coverage for the patient's mutations — this resulted
  in inconclusive outcomes in CanSeer's own demonstration
- You need synergy-based prioritization of drug combinations — the study remains limited
  in prioritizing synergy-based combinations
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: CanSeer
  role: Methodology used to annotate in silico network models with patient-specific
    genetic alterations and RNA-seq data and perform dynamical network analyses to
    obtain cell fate propensities and treatment response quantification
  repo: https://github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:edf94033a079ed44c0d6ded622b475d09302a9b4c84699fb99a32af90960c397
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# in-silico-cancer-model-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulate personalized cancer biomolecular network models by annotating a literature-derived validated network with patient-specific genetic alterations and normalized RNA-seq data, then running CanSeer's dynamical analysis (DA) to quantify cell fate propensities and treatment responses.

## When to use

Use when a validated biomolecular regulatory network model (e.g. from github.com/BIRL/CanSeer) must be annotated with patient- or cohort-specific RNA-seq/mutation data and simulated via dynamical analysis to yield cell fate propensities (proliferation, apoptosis, quiescence, senescence, cell cycle arrest).

## When NOT to use

- You need wet-lab or clinical validation of predicted efficacious drug combinations — the article notes such validation is not yet available for CanSeer predictions
- The published network model lacks coverage for the patient's mutations — this resulted in inconclusive outcomes in CanSeer's own demonstration
- You need synergy-based prioritization of drug combinations — the study remains limited in prioritizing synergy-based combinations

## Inputs

- Validated literature-derived biomolecular regulatory network model from the CanSeer repository (github.com/BIRL/CanSeer)
- Median-normalized RNA-seq gene expression values (e.g. per sample or cohort median, such as the 50 normal samples in the unpaired Case 2 LUSC use case)
- Patient-specific genetic alterations (somatic mutations, CNVs) for personalized annotation
- Drug activity scores and normalized gene expressions (optional, for drug-score re-annotation)

## Outputs

- Cell fate propensities for five fates (proliferation, apoptosis, quiescence, senescence, cell cycle arrest) reported to four decimal places
- CSV of computed propensities (e.g. case2_da_propensities.csv)
- Dynamical analysis (DA) trajectory outcomes used for treatment efficacy and cytotoxic effect computation

## How to apply

Load the validated literature-derived biomolecular regulatory network model from the CanSeer repository (github.com/BIRL/CanSeer). Prepare the annotation data, e.g. median-normalized RNA-seq gene expressions across samples (such as the 50 normal samples in the unpaired Case 2 LUSC use case) plus genetic alterations. Annotate the network model nodes with these expressions using CanSeer's model annotation step. Run CanSeer's dynamical analysis (DA) to simulate network trajectories and compute the five cell fate propensities (proliferation, apoptosis, quiescence, senescence, cell cycle arrest) from the DA outcomes. For downstream treatment assessment, drug scores (DS) computed from drug activity scores and normalized gene expressions can be used to re-annotate and re-analyze personalized models. Write the propensities to a CSV (e.g. case2_da_propensities.csv) and validate each value against reference results.

## Related tools

- **CanSeer** (Methodology used to annotate in silico network models with patient-specific genetic alterations and RNA-seq data and perform dynamical network analyses to obtain cell fate propensities and treatment response quantification) — https://github.com/BIRL/CanSeer

## Evaluation signals

- Each computed propensity matches the reported reference value to four decimal places (Case 2 median-annotated normal model: proliferation 0.1370, apoptosis 0.2709, quiescence 0.3608, senescence 0.1214, cell cycle arrest 0.0911)
- Propensity vector sums to ~1 across the five fates (0.1370+0.2709+0.3608+0.1214+0.0911 ≈ 1.0012)
- Output CSV (case2_da_propensities.csv) contains exactly five named cell fate propensity entries
- Annotation step consumed median-normalized expression values for all samples of the specified cohort (e.g. 50 normal samples, Case 2 unpaired use case)
- Dynamical analysis runs completed on the validated model without inconclusive outcomes caused by missing mutation coverage

## Limitations

- The published model used for demonstration of CanSeer lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes
- The study remains limited in prioritizing synergy-based drug combinations
- In silico predicted patient-specific drug cocktail sensitivity requires further evaluation with tumor-on-chip models
- No wet lab validation of predicted efficacious drug combinations and repositioned drugs yet
- Framework requires expansion to cater for panomics data to avoid under- or overfitting of patient's profile

## Evidence
