---
name: normal-cancer-sample-merging
description: Use when TCGA-LUSC RNA-seq gene expression data plus patient-specific
  CNV/SM/SV annotations must be split into paired (49), unpaired (50 normal / 453
  cancer), or cancer-only (453) use cases, merged, and 0-1 normalized before CanSeer
  drug-score computation.
when_to_use_negative:
- The expression matrices are already merged and 0-1 normalized (e.g., for downstream
  drug-score computation or dynamical network re-analysis) — rerunning would rescale
  values incorrectly.
- The task is annotating personalized network models or computing drug scores (DS)
  from drug activity scores and normalized expressions — those stages consume, rather
  than produce, these matrices.
- The data are not RNA-seq gene expression with corresponding patient-specific CNV/SM/SV
  annotations (e.g., panomics or whole-exome data alone), since CanSeer's normalization
  logic is defined for RNA-seq profiles.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CanSeer
  role: CanSeer's Step 2 pre-processing module implements the use-case sample selection,
    IQR/3xMAD outlier removal, N+Cs merging, and maximal-expression 0-1 normalization
    of TCGA-LUSC expression profiles
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:2e836bcd2498a06a8d6c3f21431d29989c5cfbd2d6950bee8fa48c05ce4b8bb0
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# normal-cancer-sample-merging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Skill for assembling CanSeer's three TCGA-LUSC use cases: merging normal (N) and cancer (Cs) sample expression sets per use case and normalizing each merged RNA-seq matrix to the 0-1 range by dividing by the maximal expression value, so downstream drug scores use comparably scaled gene expressions.

## When to use

Use when TCGA-LUSC RNA-seq gene expression data plus patient-specific CNV/SM/SV annotations must be split into paired (49), unpaired (50 normal / 453 cancer), or cancer-only (453) use cases, merged, and 0-1 normalized before CanSeer drug-score computation.

## When NOT to use

- The expression matrices are already merged and 0-1 normalized (e.g., for downstream drug-score computation or dynamical network re-analysis) — rerunning would rescale values incorrectly.
- The task is annotating personalized network models or computing drug scores (DS) from drug activity scores and normalized expressions — those stages consume, rather than produce, these matrices.
- The data are not RNA-seq gene expression with corresponding patient-specific CNV/SM/SV annotations (e.g., panomics or whole-exome data alone), since CanSeer's normalization logic is defined for RNA-seq profiles.

## Inputs

- TCGA-LUSC RNA-seq gene expression data (gene x sample matrix)
- Patient-specific CNV, SM (somatic mutation), and SV alteration annotations
- Sample group assignments defining the paired, unpaired, and cancer-only use cases

## Outputs

- 0-1 normalized merged expression matrix for Case 1 (49 paired samples)
- 0-1 normalized merged expression matrix for Case 2 (50 normal / 453 cancer samples)
- 0-1 normalized expression matrix for Case 3 (453 cancer-only samples)
- CSV files of the normalized matrices with CNV/SM/SV alterations tagged onto retained expression profiles

## How to apply

Load the TCGA-LUSC RNA-seq gene expression data together with patient-specific CNV, SM, and SV alteration annotations, and select samples for Case 1 paired (49), Case 2 unpaired (50 normal / 453 cancer), and Case 3 cancer-only (453). Remove outlier samples using interquartile range (IQR) and 3xMAD filtering criteria, then tag-and-store the CNV/SM/SV alterations onto the retained expression profiles so alteration information is preserved while outlier samples are dropped. Merge the normal (N) and cancer (Cs) sample sets for each use case; for the paired and unpaired cases normalize gene expressions using the highest expression in the combined N + Cs dataset, whereas in the cancer-only case normalization is performed within the cancer samples themselves. Divide each merged matrix by its maximal expression value to obtain the 0-1 range, verify reported sample counts (49, 50/453, 453), and save the matrices as CSV files.

## Related tools

- **CanSeer** (CanSeer's Step 2 pre-processing module implements the use-case sample selection, IQR/3xMAD outlier removal, N+Cs merging, and maximal-expression 0-1 normalization of TCGA-LUSC expression profiles) — github.com/BIRL/CanSeer

## Evaluation signals

- Sample counts match the article: 49 paired samples, 50 normal / 453 cancer (unpaired), and 453 cancer-only samples in the normalized matrices.
- Every normalized expression value lies in [0, 1] and the maximal expression value in each merged matrix equals exactly 1.
- For paired and unpaired cases, the normalization denominator comes from the combined N + Cs dataset; for the cancer-only case, the maximum is computed within the cancer samples only.
- CNV/SM/SV alterations are preserved (tag-and-stored) for all retained samples, while samples flagged by IQR and 3xMAD outlier filtering are absent from the matrices.
- Output matrices are saved as CSV files and re-loaded cleanly for downstream drug-score computation.

## Limitations

- Samples identified as outliers by the IQR and 3xMAD criteria are dropped along with their alteration profiles; the article notes CanSeer lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes.
- Missing CNV/SM/SV information for particular patients is not abstracted by this step; the discussion proposes integrating algorithms identifying CNVs, SMs, and SVs from whole-exome sequencing data.
- The framework requires expansion to cater for panomics data to avoid under- or overfitting of the patient's profile, so RNA-seq-only normalization may not capture all molecular variation.

## Evidence
