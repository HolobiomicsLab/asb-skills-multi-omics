---
name: machine-learning-classification-error-control
description: Use when designing a multi-omic classification study and you need the
  predicted sample size (PSS) a learner such as random forest requires to hit a target
  classification error rate (e.g., 0.01) on pilot omic feature matrices.
when_to_use_negative:
- The goal is differential feature detection rather than ML sample classification
  — MultiPower's power calculation applies there, but its significance threshold and
  effect size parameters are not applicable to classification ER control
- No pilot data are available — MultiML cannot run without prior data, and parameters
  cannot be reasonably estimated otherwise
- Input omic data are raw or still contain batch effects — MultiPower/MultiML assume
  data have been properly preprocessed and batch effects removed beforehand
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: MultiML
  role: R method that estimates the optimal sample size required by ML approaches
    (e.g., random forest) to achieve a target classification error rate on single
    or combined omic datasets
  repo: https://github.com/ConesaLab/MultiPower/blob/master/MultiPower_UsersGuide.pdf
- name: MultiPower
  role: Companion R package that hosts MultiML; installed from GitHub and complemented
    by MultiML for classification-based sample-size estimation
  repo: github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:2144029a2d3061f1c1479ccd413cc662ba3a67013d2bcf569f89a89d8feaa5c0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiML
  - MultiPower
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# machine-learning-classification-error-control

## Summary

Use the MultiML algorithm (R package bundled with MultiPower) to estimate the optimal number of samples a machine-learning classifier needs to reach a target classification error rate (ER) on one or combined omic platforms, as demonstrated on TCGA Glioblastoma subtype prediction with random forest at ER = 0.01.

## When to use

Use when designing a multi-omic classification study and you need the predicted sample size (PSS) a learner such as random forest requires to hit a target classification error rate (e.g., 0.01) on pilot omic feature matrices.

## When NOT to use

- The goal is differential feature detection rather than ML sample classification — MultiPower's power calculation applies there, but its significance threshold and effect size parameters are not applicable to classification ER control
- No pilot data are available — MultiML cannot run without prior data, and parameters cannot be reasonably estimated otherwise
- Input omic data are raw or still contain batch effects — MultiPower/MultiML assume data have been properly preprocessed and batch effects removed beforehand

## Inputs

- Preprocessed, batch-effect-free pilot omic feature matrices for each platform (e.g., TCGA Glioblastoma mRNA, miRNA, methylation, proteomics, clinical/copy-number data)
- Class labels for sample classification (e.g., GBM tumor subtypes)
- Target classification error rate (e.g., 0.01) and choice of ML classifier (random forest)

## Outputs

- Table of predicted sample sizes (PSS) per omic-platform combination at the target ER (e.g., multiml_gbm_sample_sizes.csv)

## How to apply

Load preprocessed, batch-effect-free pilot omic feature matrices (e.g., mRNA, miRNA, methylation, proteomics, clinical/copy-number from TCGA Glioblastoma) into R — pilot data are a requisite for MultiML. Install MultiPower from GitHub (devtools::install_github('ConesaLab/MultiPower')) and read MultiML's User Guide (MultiPower_UsersGuide.pdf) for run instructions. Configure MultiML with random forest as the classifier and set the target classification error rate to 0.01, then run the estimation procedure separately for each combination of two, three, four, and five omic platforms. Collect the predicted sample sizes (PSS) per platform combination into a single table and export it (e.g., multiml_gbm_sample_sizes.csv). The rationale is that MultiML learns from pilot data how classification ER decreases with n, allowing sample-size planning for ML classification where classical MultiPower power parameters (significance threshold, effect size) do not apply.

## Related tools

- **MultiML** (R method that estimates the optimal sample size required by ML approaches (e.g., random forest) to achieve a target classification error rate on single or combined omic datasets) — https://github.com/ConesaLab/MultiPower/blob/master/MultiPower_UsersGuide.pdf
- **MultiPower** (Companion R package that hosts MultiML; installed from GitHub and complemented by MultiML for classification-based sample-size estimation) — github.com/ConesaLab/MultiPower

## Examples

```
install.packages('devtools'); devtools::install_github('ConesaLab/MultiPower')
```

## Evaluation signals

- The output table contains exactly one PSS value per platform combination (2-, 3-, 4-, and 5-omic) for the ER = 0.01 target
- PSS values are monotonically non-increasing as the number of combined omics grows, matching the reported complementary multi-omic behaviour (Fig. 5b-d)
- MultiML was configured with random forest as the classifier and target ER 0.01, and only preprocessed pilot data were supplied
- The final PSS table was written to a single CSV (e.g., multiml_gbm_sample_sizes.csv) suitable for experimental-design comparison

## Limitations

- MultiML requires pilot data; it cannot run without prior data
- MultiPower's power parameters (significance threshold, effect size) are not applicable to ML sample classification, which is why MultiML uses classification error rate as the performance measure
- Input data must be properly preprocessed and batch effects removed before applying MultiPower/MultiML — the tool does not handle this
- MultiPower/MultiML parameter estimates are optimally computed from pilot data; without pilot data users must set parameters manually

## Evidence

- [abstract] MultiPower is complemented with MultiML, an algorithm to estimate sample size for machine learning classification problems based on multi-omic data: "MultiPower is complemented with MultiML, an algorithm to estimate sample size for machine learning classification problems based on multi-omic data"
- [intro] MultiML R method intro: "we present MultiML, an R method to obtain the optimal sample size required by ML approaches to achieve a target classification ER"
- [discussion] Pilot data requisite: "Hence, pilot data are a requisite for MultiML."
- [methods] Classification error rate control workflow: "help users determine the optimal sample size required to control for classification error rates when using one or more omics datasets"
- [readme] MultiML in README: "The MultiML method is included as a complementary tool to MultiPower"
