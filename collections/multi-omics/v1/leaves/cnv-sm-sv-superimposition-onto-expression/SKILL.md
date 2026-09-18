---
name: cnv-sm-sv-superimposition-onto-expression
description: Use when processing TCGA-LUSC RNA-seq gene expression data with paired
  patient-specific CNV/SM/SV alteration annotations, and you must preserve alterations
  while filtering outlier samples and normalizing expression per use case.
when_to_use_negative:
- Input data lacks patient-specific CNV/SM/SV alteration annotations paired to the
  expression samples, since the superimposition step cannot preserve alteration information
- The goal is downstream drug-score computation or dynamical network analysis rather
  than Step 2 omics pre-processing and normalization
- Expression data comes from a non-LUSC cohort or from whole-exome sequencing without
  matched RNA-seq profiles, which the TCGA-LUSC-specific sample counts and filtering
  criteria were not designed for
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: CanSeer
  role: 'Methodology implementing the Step 2 pre-processing: use-case sample selection,
    IQR/3xMAD outlier filtering, CNV/SM/SV superimposition, and 0-1 max-value expression
    normalization'
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:5058fb61c1e48180cc96efc50943e6d6af841a3056d0280356316339ab405ece
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# cnv-sm-sv-superimposition-onto-expression

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A TCGA-LUSC omics pre-processing step in CanSeer's Step 2 that tags-and-stores patient-specific CNV, SM, and SV alterations onto retained RNA-seq gene expression profiles, then normalizes merged normal+cancer expression matrices to the 0-1 range by dividing by the maximal expression value.

## When to use

Use when processing TCGA-LUSC RNA-seq gene expression data with paired patient-specific CNV/SM/SV alteration annotations, and you must preserve alterations while filtering outlier samples and normalizing expression per use case.

## When NOT to use

- Input data lacks patient-specific CNV/SM/SV alteration annotations paired to the expression samples, since the superimposition step cannot preserve alteration information
- The goal is downstream drug-score computation or dynamical network analysis rather than Step 2 omics pre-processing and normalization
- Expression data comes from a non-LUSC cohort or from whole-exome sequencing without matched RNA-seq profiles, which the TCGA-LUSC-specific sample counts and filtering criteria were not designed for

## Inputs

- TCGA-LUSC RNA-seq gene expression data
- Patient-specific CNV (copy number variation) annotations
- Patient-specific SM (somatic mutation) annotations
- Patient-specific SV (structural variation) annotations
- Use-case sample assignments: paired (49), unpaired (50 normal / 453 cancer), cancer-only (453)

## Outputs

- Normalized 0-1 expression matrices per use case, saved as CSV files
- Tag-and-store alteration annotations (CNV/SM/SV) mapped onto retained expression profiles
- Verified sample counts: 49 paired, 50 normal / 453 cancer, 453 cancer-only

## How to apply

Load the TCGA-LUSC RNA-seq gene expression data together with patient-specific CNV, SM, and SV alteration annotations, then select samples for the three use cases: Case 1 paired samples (49), Case 2 unpaired samples (50 normal / 453 cancer), and Case 3 cancer-only samples (453). Remove outlier samples using the interquartile range (IQR) and 3xMAD filtering criteria. Tag-and-store the CNV/SM/SV alterations superimposed onto the retained expression profiles so alteration information is preserved while outlier samples are dropped. Merge the normal (N) and cancer (Cs) sample sets for each use case, then normalize each merged expression matrix to the 0-1 range by dividing by the maximal expression value; for the paired and unpaired cases the maximum is taken over the combined N + CS dataset, whereas for the cancer-only case normalization is performed within the cancer samples themselves. Save the normalized matrices as CSV files.

## Related tools

- **CanSeer** (Methodology implementing the Step 2 pre-processing: use-case sample selection, IQR/3xMAD outlier filtering, CNV/SM/SV superimposition, and 0-1 max-value expression normalization) — github.com/BIRL/CanSeer

## Evaluation signals

- Each normalized matrix contains the reported sample counts: 49 paired (Case 1), 50 normal / 453 cancer (Case 2), and 453 cancer-only (Case 3)
- All values in each normalized expression matrix fall within the 0-1 range after division by the maximal expression value
- CNV/SM/SV alteration annotations are present for all retained samples and absent for dropped outlier samples (tag-and-store preserved)
- Paired and unpaired matrices were normalized using the maximum over the combined normal+cancer (N + CS) dataset, while the cancer-only matrix was normalized within cancer samples
- Output matrices are saved as CSV files without NaNs introduced by the merge step

## Limitations

- Missing CNV/SM/SV information for particular patients is not yet abstracted; the article notes that algorithms for identifying CNVs, SMs, and SVs could be integrated to abstract missing information from whole-exome sequencing data
- Outlier removal via IQR and 3xMAD filtering may drop samples and their alteration annotations, reducing the effective patient set available for personalized modeling
- Normalization to the combined N + CS maximum differs between use cases, so expression values are not directly comparable across the paired/unpaired and cancer-only normalized matrices

## Evidence
