---
name: expression-outlier-filtering-iqr-mad
description: Use when raw TCGA-LUSC RNA-seq expression matrices (paired, unpaired,
  or cancer-only cohorts) must be quality-filtered for outliers before normalization
  and in silico model annotation.
when_to_use_negative:
- Input expression data have already been IQR/3xMAD outlier-filtered and normalized
  — reapplying the cutoffs would drop valid samples
- The cohort is not LUSC RNA-seq data with paired CNV/SM/SV annotations, so the CanSeer
  use-case sample structure (49 paired, 50/453 unpaired, 453 cancer-only) does not
  apply
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: CanSeer
  role: 'Implements the TCGA-LUSC omics pre-processing module: IQR/3xMAD outlier filtering,
    N+Cs sample merging, max-based 0-1 normalization, and CSV export of the use-case
    expression matrices used to compute drug scores'
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:4ee0fb8160412f8eedd7d4ddcba668a46733ca379c11b599c7fb0601b391c076
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Expression outlier filtering (IQR and 3xMAD)

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A pre-processing skill from CanSeer's TCGA-LUSC omics module that removes outlier RNA-seq samples using interquartile range (IQR) and 3xMAD criteria while preserving patient-specific CNV/SM/SV alteration annotations, before 0-1 max-based normalization of normal (N), cancer (Cs), and combined N+Cs expression matrices.

## When to use

Use when raw TCGA-LUSC RNA-seq expression matrices (paired, unpaired, or cancer-only cohorts) must be quality-filtered for outliers before normalization and in silico model annotation.

## When NOT to use

- Input expression data have already been IQR/3xMAD outlier-filtered and normalized — reapplying the cutoffs would drop valid samples
- The cohort is not LUSC RNA-seq data with paired CNV/SM/SV annotations, so the CanSeer use-case sample structure (49 paired, 50/453 unpaired, 453 cancer-only) does not apply

## Inputs

- TCGA-LUSC RNA-seq gene expression matrix (raw counts/TPM) for normal (N) and cancer (Cs) samples
- Patient-specific CNV, SM (somatic mutation), and SV alteration annotations keyed to the expression samples
- Use-case sample selections: paired (49), unpaired (50 normal / 453 cancer), cancer-only (453)

## Outputs

- Outlier-filtered, 0-1 normalized expression matrices (one per use case) saved as CSV files
- Retained CNV/SM/SV alteration annotations superimposed onto the filtered expression profiles

## How to apply

Load the TCGA-LUSC RNA-seq gene expression data together with patient-specific CNV, SM, and SV alteration annotations, and select the samples for the three use cases: Case 1 paired samples (49), Case 2 unpaired samples (50 normal / 453 cancer), and Case 3 cancer-only samples (453). Remove outlier samples using the interquartile range (IQR) and 3xMAD filtering criteria, taking care to tag-and-store the CNV/SM/SV alterations superimposed onto the retained expression profiles so alteration information is preserved while outlier samples are dropped. Merge the normal (N) and cancer (Cs) sample sets per use case, then normalize each merged matrix to the 0-1 range by dividing by the maximal expression value — for paired and unpaired cases the maximum is taken over the combined N + Cs dataset, whereas for the cancer-only case normalization is performed within the cancer samples themselves. Save each filtered, normalized matrix as a CSV file, checking that the reported sample counts (49 paired; 50 normal / 453 cancer; 453 cancer-only) are retained.

## Related tools

- **CanSeer** (Implements the TCGA-LUSC omics pre-processing module: IQR/3xMAD outlier filtering, N+Cs sample merging, max-based 0-1 normalization, and CSV export of the use-case expression matrices used to compute drug scores) — github.com/BIRL/CanSeer

## Evaluation signals

- Filtered and merged matrices contain the reported sample counts: 49 paired, 50 normal / 453 cancer, and 453 cancer-only samples
- Every normalized expression value lies in the 0-1 range, with each matrix maximum equal to 1 (dividing by the maximal expression value)
- Paired and unpaired matrices are normalized against the maximum of the combined N + Cs dataset, while the cancer-only matrix is normalized within cancer samples — verify the normalization reference differs per use case
- CNV/SM/SV alteration records are present for every retained patient sample; no alteration annotations remain for dropped outlier samples
- Output matrices are saved as CSV files with sample identifiers matching the input TCGA-LUSC expression table

## Limitations

- The published model used for demonstration of CanSeer lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes
- The framework requires an expansion to cater for panomics data, to avoid under or overfitting of a patient's profile
- Missing CNV/SM/SV information for particular patients is not yet abstracted; algorithms identifying CNVs, SMs, and SVs could be integrated to abstract the missing information from whole-exome sequencing data

## Evidence

- [methods] Filtering criteria: [evidence span withheld — anchor status: not_found]
- [methods] Normalization scheme: [evidence span withheld — anchor status: not_found]
