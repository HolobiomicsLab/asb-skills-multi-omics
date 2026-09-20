---
name: tcga-rnaseq-sample-cohort-selection
description: Use when building TCGA-LUSC expression matrices for paired (normal+cancer),
  unpaired, or cancer-only analyses, when outlier removal (IQR/3xMAD) and max-value
  0-1 normalization are required before model annotation.
when_to_use_negative:
- Expression data is already normalized or comes from a cohort other than the CanSeer
  paired/unpaired/cancer-only TCGA-LUSC use cases
- Analysis needs panomics or multi-omics integration rather than RNA-seq expression
  matrices — the paper flags that the framework 'requires an expansion to cater for
  panomics data'
- A specific patient lacks CNV/SM/SV information and complete alteration overlays
  are mandatory, since missing alteration information is not yet abstracted within
  CanSeer
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2648
tools:
- name: CanSeer
  role: 'Implements the TCGA-LUSC pre-processing module: use-case sample selection,
    IQR/3xMAD outlier filtering, alteration tag-and-store, and max-value 0-1 expression
    normalization'
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:dd155e34064dee86a0ea60dd1f357ac27751cdd456b4d8e43c64fc5883f208b7
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
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2648
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# tcga-rnaseq-sample-cohort-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects TCGA-LUSC RNA-seq patient cohorts into CanSeer's three use cases (paired, unpaired, cancer-only), removes outlier samples with IQR and 3xMAD filtering, and normalizes gene expression to the 0-1 range so downstream drug-score computation uses comparable expression profiles.

## When to use

Use when building TCGA-LUSC expression matrices for paired (normal+cancer), unpaired, or cancer-only analyses, when outlier removal (IQR/3xMAD) and max-value 0-1 normalization are required before model annotation.

## When NOT to use

- Expression data is already normalized or comes from a cohort other than the CanSeer paired/unpaired/cancer-only TCGA-LUSC use cases
- Analysis needs panomics or multi-omics integration rather than RNA-seq expression matrices — the paper flags that the framework 'requires an expansion to cater for panomics data'
- A specific patient lacks CNV/SM/SV information and complete alteration overlays are mandatory, since missing alteration information is not yet abstracted within CanSeer

## Inputs

- TCGA-LUSC RNA-seq gene expression matrix
- Patient-specific CNV alteration annotations
- Patient-specific SM (somatic mutation) annotations
- Patient-specific SV (structural variant) annotations
- Use-case definition (paired / unpaired / cancer-only) with expected sample counts

## Outputs

- Normalized 0-1 expression matrix per use case (CSV), e.g. 49 paired, 50 normal / 453 cancer, 453 cancer-only samples
- Retained sample list after IQR/3xMAD outlier filtering
- Tagged CNV/SM/SV alteration overlays aligned to the retained expression profiles

## How to apply

Load the TCGA-LUSC RNA-seq gene expression data together with patient-specific CNV, SM, and SV alteration annotations, then partition samples into the three use cases: Case 1 paired samples (49), Case 2 unpaired samples (50 normal / 453 cancer), and Case 3 cancer-only samples (453). Remove outlier samples using the interquartile range (IQR) and 3xMAD filtering criteria, tagging-and-storing the CNV/SM/SV alterations superimposed on the retained expression profiles so alteration information is preserved while outliers are dropped. Merge the normal (N) and cancer (Cs) sample sets per use case. Normalize each merged expression matrix to the 0-1 range by dividing by the maximal expression value; for the paired and unpaired cases the maximum is taken over the combined normal+cancer (N + CS) dataset, whereas in the cancer-only case normalization is performed within the cancer samples themselves. Save each normalized matrix as a CSV file.

## Related tools

- **CanSeer** (Implements the TCGA-LUSC pre-processing module: use-case sample selection, IQR/3xMAD outlier filtering, alteration tag-and-store, and max-value 0-1 expression normalization) — github.com/BIRL/CanSeer

## Evaluation signals

- Each normalized matrix contains the reported sample counts: 49 paired, 50 normal / 453 cancer, 453 cancer-only
- All expression values in each output matrix fall within the 0-1 range after division by the maximal expression value
- Outlier removal is reproducible: dropped samples correspond to the IQR and 3xMAD filtering criteria
- CNV/SM/SV alterations are still present for every retained sample after outlier filtering (tag-and-store preserved the overlays)
- Paired and unpaired matrices are normalized against the same combined N + CS maximum, while the cancer-only matrix uses a cancer-samples-internal maximum

## Limitations

- Missing CNV/SM/SV information for particular patients is not yet abstracted; algorithms identifying CNVs, SMs, and SVs would need to be integrated
- The framework requires an expansion to cater for panomics data, to avoid under- or over-fitting of a patient's profile
- Sample counts are cohort-specific (49 paired, 50 normal / 453 cancer, 453 cancer-only); other TCGA cohorts or updated TCGA releases will not match these counts

## Evidence

- [abstract] Three use cases of lung squamous cell carcinoma (LUSC) patients are provided involving paired samples, unpaired samples, and cancer samples only: [evidence span withheld — anchor status: not_found]
- [methods] Remove outlier samples using the interquartile range (IQR) and 3xMAD filtering criteria.: [evidence span withheld — anchor status: not_found]
- [results] normalizes gene expressions using the highest expression in the combined normal+cancer (N + CS) dataset: [evidence span withheld — anchor status: not_found]
- [methods] Verify that each normalized matrix contains the reported sample counts (49 paired, 50 normal / 453 cancer, 453 cancer-only) and save the matrices as CSV files.: [evidence span withheld — anchor status: not_found]
