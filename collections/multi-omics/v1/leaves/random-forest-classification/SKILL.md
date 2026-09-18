---
name: random-forest-classification
description: Use when designing a multi-omic classification experiment and you need
  to know how many samples a random forest predictor needs to achieve a fixed target
  error rate across single- and multi-omic feature combinations.
when_to_use_negative:
- The goal is differential feature analysis power (sample size for detecting differential
  features) rather than ML sample classification — use MultiPower's power calculation
  instead
- No pilot data are available — MultiML requires prior pilot data and cannot run without
  it
- The input data have unremoved batch effects or are otherwise unpreprocessed — MultiML/MultiPower
  do not handle preprocessing
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MultiML
  role: Estimates the optimal sample size required by random forest classification
    to achieve a target classification error rate from multi-omic pilot data
  repo: github.com/ConesaLab/MultiPower
- name: MultiPower
  role: R package (installed from GitHub) that hosts MultiML; MultiPower itself is
    not directly applicable to ML classification since significance threshold and
    effect size do not apply
  repo: github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:70e0239ed458c5d0d9bb45826bcde39694db5b2c2a15874595fa344bd761e3ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/MultiPower
  tools:
  - MultiML
  - MultiPower
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# random-forest-classification

## Summary

Applies random forest as the classifier within the MultiML framework to estimate the sample size (PSS) required to reach a target classification error rate (e.g. ER=0.01) when predicting sample labels (e.g. TCGA Glioblastoma subtypes) from one or combined omic platforms.

## When to use

Use when designing a multi-omic classification experiment and you need to know how many samples a random forest predictor needs to achieve a fixed target error rate across single- and multi-omic feature combinations.

## When NOT to use

- The goal is differential feature analysis power (sample size for detecting differential features) rather than ML sample classification — use MultiPower's power calculation instead
- No pilot data are available — MultiML requires prior pilot data and cannot run without it
- The input data have unremoved batch effects or are otherwise unpreprocessed — MultiML/MultiPower do not handle preprocessing

## Inputs

- Preprocessed, batch-effect-free pilot omic feature matrices (R data), one per platform (e.g. mRNA, miRNA, methylation, proteomics, clinical/copy-number)
- Sample class labels (e.g. TCGA Glioblastoma tumor subtypes)
- Target classification error rate (e.g. ER = 0.01)

## Outputs

- Predicted optimal sample size (PSS) per random forest classifier per omic platform combination at the target ER
- Sample-size table across single- and multi-omic combinations (e.g. multiml_gbm_sample_sizes.csv)

## How to apply

Load preprocessed pilot omic feature matrices (e.g. TCGA Glioblastoma mRNA, miRNA, methylation, proteomics, clinical/copy-number) into R; pilot data are a requisite for MultiML and must be batch-effect free before use. Install MultiPower from GitHub (devtools::install_github('ConesaLab/MultiPower')) and configure MultiML with random forest as the classifier and a target classification error rate of 0.01. Run the MultiML estimation separately for each combination of two, three, four, and five omic platforms, letting the algorithm learn from the pilot data. Collect the predicted optimal sample sizes (PSS) per platform combination and verify they decrease monotonically as more omics are combined, reflecting complementary multi-omic information. Note that MultiPower alone cannot be used here because significance threshold and effect size are not applicable to ML classification; MultiML uses classification error rate as the performance measure instead.

## Related tools

- **MultiML** (Estimates the optimal sample size required by random forest classification to achieve a target classification error rate from multi-omic pilot data) — github.com/ConesaLab/MultiPower
- **MultiPower** (R package (installed from GitHub) that hosts MultiML; MultiPower itself is not directly applicable to ML classification since significance threshold and effect size do not apply) — github.com/ConesaLab/MultiPower

## Examples

```
install.packages("devtools"); devtools::install_github("ConesaLab/MultiPower")
```

## Evaluation signals

- Output table contains exactly one PSS value per platform combination for the ER=0.01 target
- PSS values are monotonically non-increasing as the number of combined omic platforms grows (2 → 5 platforms), matching reported Fig. 5b-d behaviour
- Each PSS estimate was derived from the supplied pilot data, not manual parameter entry
- Input matrices were preprocessed and batch-effect free before running MultiML
- The classifier is random forest and the performance measure used is classification error rate, not significance threshold/effect size

## Limitations

- MultiML requires pilot data; it cannot run without prior data
- MultiPower is not directly applicable to ML methods for sample classification because significance threshold and effect size are not applicable parameters
- Input data must be properly preprocessed and batch effects removed before applying the method; no handling is provided within the tool

## Evidence

- [abstract] MultiML abstract: "MultiML, an algorithm to estimate sample size for machine learning classification problems based on multi-omic data"
- [methods] MultiML workflow step: "help users determine the optimal sample size required to control for classification error rates when using one or more omics datasets"
- [methods] MultiML installation: "devtools::install_github(“ConesaLab/MultiPower”)"
- [discussion] pilot data requirement: "Hence, pilot data are a requisite for MultiML."
- [discussion] MultiML not applicable via MultiPower: "The MultiPower approach is not directly applicable to ML methods used for sample classification, as in this case the basic parameters of the power calculation—significance threshold and effect"
- [readme] README MultiML description: "The MultiML method is included as a complementary tool to MultiPower, and is designed to help users determine the optimal sample size required to control for classification error rates"
