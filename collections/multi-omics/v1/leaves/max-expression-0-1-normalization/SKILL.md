---
name: max-expression-0-1-normalization
description: 'Use when preprocessing raw TCGA-LUSC RNA-seq matrices for CanSeer: you
  have merged normal (N) and cancer (Cs) sample sets per use case and need 0-1 scaled
  expressions that feed patient-specific drug score (DS) computation.'
when_to_use_negative:
- Expressions are already normalized against a shared external reference (e.g. TPM/size-factor
  normalized for cross-dataset comparison) — per-dataset max scaling would destroy
  comparability between use cases
- You need to directly compare magnitudes across the cancer-only and paired/unpaired
  use cases, since the cancer-only case is normalized within cancer samples while
  paired/unpaired use the combined N + Cs maximum
- The input is a single-sample patient profile for downstream dynamical analysis rather
  than a cohort expression matrix to be scaled
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CanSeer
  role: Methodology within which the 0-1 max-expression normalization is performed
    on merged N + Cs TCGA-LUSC matrices before patient-specific drug score computation
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:cc77488ec16406a7b9b8f1148d3f8d41c5e52492fd718e34d9b78f23947240e6
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
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Max-expression 0-1 normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A gene-expression normalization scheme from CanSeer in which merged normal+cancer (N + Cs) RNA-seq matrices are rescaled to the 0-1 range by dividing every expression value by the maximal expression value; for paired and unpaired LUSC use cases the max is taken over the combined N + Cs dataset, while for the cancer-only case it is computed within the cancer samples themselves.

## When to use

Use when preprocessing raw TCGA-LUSC RNA-seq matrices for CanSeer: you have merged normal (N) and cancer (Cs) sample sets per use case and need 0-1 scaled expressions that feed patient-specific drug score (DS) computation.

## When NOT to use

- Expressions are already normalized against a shared external reference (e.g. TPM/size-factor normalized for cross-dataset comparison) — per-dataset max scaling would destroy comparability between use cases
- You need to directly compare magnitudes across the cancer-only and paired/unpaired use cases, since the cancer-only case is normalized within cancer samples while paired/unpaired use the combined N + Cs maximum
- The input is a single-sample patient profile for downstream dynamical analysis rather than a cohort expression matrix to be scaled

## Inputs

- TCGA-LUSC RNA-seq gene expression matrix (raw counts/expressions per gene per sample)
- Patient-specific CNV, SM (somatic mutation), and SV alteration annotations to be tag-and-stored alongside retained samples
- Use-case sample assignments (paired, unpaired normal/cancer, cancer-only) after IQR + 3xMAD outlier filtering

## Outputs

- Gene expression matrix normalized to the 0-1 range by division by the maximal expression value, per use case
- Saved CSV files of the normalized matrices (49 paired; 50 normal / 453 cancer; 453 cancer-only)
- Normalized expressions suitable for CanSeer drug score (DS) computation

## How to apply

First assemble the three LUSC use-case matrices: paired (49 samples), unpaired (50 normal / 453 cancer), and cancer-only (453), after removing outlier samples via interquartile-range (IQR) and 3xMAD filtering and tag-and-storing the CNV/SM/SV alterations onto retained expression profiles. Merge the normal (N) and cancer (Cs) sets for each use case, then divide the whole merged expression matrix by its maximal expression value so all values lie in the 0-1 range. The decision point is the normalization denominator: for paired and unpaired cases use the highest expression in the combined N + Cs dataset, whereas for the cancer-only case normalize within the cancer samples only. Save each normalized matrix as a CSV and verify the reported sample counts (49 paired; 50 normal / 453 cancer; 453 cancer-only). These 0-1 scaled expressions are the ones CanSeer later combines with drug activity scores to compute per-patient drug scores (DS).

## Related tools

- **CanSeer** (Methodology within which the 0-1 max-expression normalization is performed on merged N + Cs TCGA-LUSC matrices before patient-specific drug score computation) — github.com/BIRL/CanSeer

## Evaluation signals

- Every value in each output matrix lies within [0, 1] and the maximum expression value equals 1.0
- Sample counts match the reported use cases: 49 paired, 50 normal / 453 cancer, and 453 cancer-only
- For paired and unpaired cases the normalization denominator is the highest expression in the combined N + Cs dataset; for cancer-only it is the max within the cancer samples
- CNV/SM/SV alteration tags are preserved on retained samples after IQR and 3xMAD outlier removal
- Each normalized matrix is persisted as a CSV file

## Limitations

- Normalization denominators differ per use case (combined N + Cs max vs. cancer-only max), so absolute expression values are not directly comparable across the three use cases
- Outlier removal via IQR and 3xMAD changes the samples entering the max calculation, so filtering choices propagate into the normalized scale
- More broadly, CanSeer requires expansion to panomics data to avoid under- or overfitting of a patient's profile, and missing CNV/SM/SV information for particular patients is not yet abstracted

## Evidence
