---
name: omics-pilot-data-preparation
description: Use when running MultiPower or MultiML sample-size/power estimation,
  since both algorithms require properly preprocessed pilot datasets per omic platform
  before the estimation procedure can run.
when_to_use_negative:
- Data still contain batch effects or are otherwise not properly preprocessed, since
  MultiPower/MultiML do not handle preprocessing internally
- No prior/pilot data are available for the platforms, since pilot data are a requisite
  for MultiML and manual parameters would otherwise be needed
- The goal is power calculation for differential features analysis via MultiPower
  while significance threshold and effect size parameters are not applicable (e.g.
  ML classification sample-size questions should use MultiML instead)
tools:
- name: MultiPower
  role: R package that consumes the preprocessed pilot multi-omic matrices to estimate
    optimal sample size in a multi-omics experiment
  repo: github.com/ConesaLab/MultiPower
- name: MultiML
  role: Complementary algorithm that uses the pilot data to estimate sample size for
    machine learning classification problems across one or more omics datasets
  repo: github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:798c848de9e28fdff1dc8939c665af20ddcaf19703a0a13ab270426e19f1fc4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/MultiPower
  tools:
  - MultiML
  - MultiPower
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# omics-pilot-data-preparation

## Summary

Prepares preprocessed, batch-effect-free multi-omic pilot matrices (e.g. TCGA Glioblastoma mRNA, miRNA, methylation, proteomics, clinical/copy-number) as the mandatory prior-data input required by MultiPower and MultiML to estimate optimal sample sizes.

## When to use

Use when running MultiPower or MultiML sample-size/power estimation, since both algorithms require properly preprocessed pilot datasets per omic platform before the estimation procedure can run.

## When NOT to use

- Data still contain batch effects or are otherwise not properly preprocessed, since MultiPower/MultiML do not handle preprocessing internally
- No prior/pilot data are available for the platforms, since pilot data are a requisite for MultiML and manual parameters would otherwise be needed
- The goal is power calculation for differential features analysis via MultiPower while significance threshold and effect size parameters are not applicable (e.g. ML classification sample-size questions should use MultiML instead)

## Inputs

- Preprocessed omic feature matrices per platform (e.g. mRNA, miRNA, methylation, proteomics, clinical/copy-number) from a shared cohort (pilot data)
- Batch-effect-free data (cleaned upstream, not by MultiPower/MultiML)

## Outputs

- Pilot datasets loaded in R ready for MultiPower/MultiML estimation
- Downstream: predicted optimal sample size (PSS) tables per platform combination at a given target error rate (e.g. multiml_gbm_sample_sizes.csv)

## How to apply

Collect each omic platform (e.g. two to five platforms such as mRNA, miRNA, methylation, proteomics, and clinical/copy-number data, as in the TCGA Glioblastoma subtype-prediction study) as a feature matrix with samples in the same cohort. Ensure the data are properly preprocessed and that any batch effects are removed beforehand, because MultiPower and MultiML provide no internal handling for this. Note that MultiML learns from pilot data as a requisite: without prior data it cannot run, and MultiPower parameters that are not estimated from pilot data must be manually provided. In R, install the ConesaLab/MultiPower package via devtools and its dependencies (FDRsampsize, lpSolve) before loading the pilot matrices into the MultiPower/MultiML functions. For MultiML, configure the classifier (e.g. random forest) and the target classification error rate (e.g. ER = 0.01) before running the estimation over each platform combination.

## Related tools

- **MultiPower** (R package that consumes the preprocessed pilot multi-omic matrices to estimate optimal sample size in a multi-omics experiment) — github.com/ConesaLab/MultiPower
- **MultiML** (Complementary algorithm that uses the pilot data to estimate sample size for machine learning classification problems across one or more omics datasets) — github.com/ConesaLab/MultiPower

## Examples

```
install.packages("devtools"); devtools::install_github("ConesaLab/MultiPower")
```

## Evaluation signals

- Each pilot matrix is confirmed batch-effect free and properly preprocessed before loading into MultiPower/MultiML
- The pilot data cover the same sample cohort across all omic platforms intended for the combination (two to five platforms in the TCGA GBM case)
- MultiML runs without error, confirming the pilot-data requisite is satisfied; missing parameters that would otherwise need manual provision are accounted for
- Downstream PSS output contains one value per platform combination at the chosen target (e.g. ER = 0.01) and values are monotonically non-increasing as more omics are combined

## Limitations

- MultiML requires pilot data and cannot run without prior data
- MultiPower parameter estimates depend on pilot data availability; otherwise users must set parameters manually
- Input data must be preprocessed and batch-effect free before applying MultiPower; no handling is provided within the tool
- MultiPower is not directly applicable to ML methods for sample classification because significance threshold and effect size are not applicable in that setting

## Evidence

- [discussion] Data should have been properly preprocessed and eventual batch effects, removed.: "Data should have been properly preprocessed and eventual batch effects, removed."
- [discussion] Hence, pilot data are a requisite for MultiML.: "Hence, pilot data are a requisite for MultiML."
- [methods] devtools::install_github(“ConesaLab/MultiPower”): "devtools::install_github(“ConesaLab/MultiPower”)"
- [abstract] MultiML, an algorithm to estimate sample size for machine learning classification problems based on multi-omic data: "MultiML, an algorithm to estimate sample size for machine learning classification problems based on multi-omic data"
- [readme] Some dependencies are required before running MultiPower that can be installed from R via install.packages(): "Some dependencies are required before running MultiPower that can be installed from R via 
`install.packages()` and loaded with `library()`:"
