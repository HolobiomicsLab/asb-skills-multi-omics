---
name: sample-size-curve-interpretation
description: Use when MultiML (or similar) has produced predicted sample sizes (PSS)
  per omic-platform combination at a fixed target ER and you must validate or interpret
  the sample-size trend before design decisions.
when_to_use_negative:
- No pilot data are available — MultiML cannot run without prior data, so there is
  no PSS curve to interpret
- The goal is power/sample size for differential feature analysis rather than ML sample
  classification — use MultiPower instead, since significance threshold and effect
  size are not applicable to classification ER
- Input omic matrices are raw or still contain batch effects — MultiML requires properly
  preprocessed data, so interpreting its output would be invalid
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3577
tools:
- name: MultiML
  role: Estimates the optimal sample size (PSS) required by ML classifiers (e.g.,
    random forest) to achieve a target classification error rate on one or more omic
    datasets; its per-combination outputs are the object of interpretation
  repo: https://github.com/ConesaLab/MultiPower/blob/master/MultiPower_UsersGuide.pdf
- name: MultiPower
  role: Companion R package (installed from GitHub) that ships MultiML and provides
    the power-calculation framework in which classification-error-based sample-size
    results are contextualized
  repo: github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:9d137a472639bd2c7c5423d3ba15016e9aa87240c37ab912512cf786f225cbf0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiML
  - MultiPower
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# sample-size-curve-interpretation

## Summary

Interprets MultiML-predicted sample-size (PSS) results as a function of the number of combined omics platforms and the target classification error rate (ER), verifying that predicted sample sizes decrease as complementary multi-omic information is added.

## When to use

Use when MultiML (or similar) has produced predicted sample sizes (PSS) per omic-platform combination at a fixed target ER and you must validate or interpret the sample-size trend before design decisions.

## When NOT to use

- No pilot data are available — MultiML cannot run without prior data, so there is no PSS curve to interpret
- The goal is power/sample size for differential feature analysis rather than ML sample classification — use MultiPower instead, since significance threshold and effect size are not applicable to classification ER
- Input omic matrices are raw or still contain batch effects — MultiML requires properly preprocessed data, so interpreting its output would be invalid

## Inputs

- Preprocessed, batch-effect-free TCGA Glioblastoma omic feature matrices (two to five platforms, e.g., mRNA, miRNA, methylation, proteomics, clinical/copy-number) used as MultiML pilot data
- MultiML output: predicted optimal sample sizes (PSS) for each platform combination at a fixed target classification error rate (e.g., ER=0.01)

## Outputs

- Sample-size table (e.g., multiml_gbm_sample_sizes.csv) with one PSS value per omic-platform combination at the target ER
- Interpretation/validation statement that PSS is monotonically non-increasing with the number of combined omics

## How to apply

Run MultiML with a fixed classifier (e.g., random forest) and a fixed target classification error rate (e.g., ER=0.01) over each combination of two, three, four, and five omic platforms (e.g., mRNA, miRNA, methylation, proteomics, clinical/copy-number from TCGA Glioblastoma). Collect the predicted optimal sample size (PSS) for every platform combination into a single table. Interpret the curve: a correct result shows PSS monotonically non-increasing as more omics are added, because complementary multi-omic information improves predictor efficiency (cf. Fig. 5b-d behaviour in the paper). Judge correctness by (i) exactly one PSS value per platform combination at the chosen ER, and (ii) the non-increasing monotonicity of PSS with platform count; deviations suggest preprocessing or pilot-data problems.

## Related tools

- **MultiML** (Estimates the optimal sample size (PSS) required by ML classifiers (e.g., random forest) to achieve a target classification error rate on one or more omic datasets; its per-combination outputs are the object of interpretation) — https://github.com/ConesaLab/MultiPower/blob/master/MultiPower_UsersGuide.pdf
- **MultiPower** (Companion R package (installed from GitHub) that ships MultiML and provides the power-calculation framework in which classification-error-based sample-size results are contextualized) — github.com/ConesaLab/MultiPower

## Examples

```
devtools::install_github("ConesaLab/MultiPower"); # then run MultiML per platform combination with classifier=random forest and target ER=0.01 (see MultiPower_UsersGuide.pdf)
```

## Evaluation signals

- Output table contains exactly one PSS value per omic-platform combination (two through five platforms) at the fixed target ER (0.01)
- PSS values are monotonically non-increasing as the number of combined omics grows, matching the reported Fig. 5b-d behaviour
- All PSS estimates derive from the same classifier (random forest) and the same target ER, so differences across combinations are attributable to platform count only
- Pilot input matrices were preprocessed and batch-effect removed before MultiML ran, as the paper requires
- Each PSS estimate is accompanied by the platform list it was computed from, allowing traceable comparison against the paper's TCGA Glioblastoma subtype-prediction setting

## Limitations

- MultiML requires pilot data; it cannot run without prior data (Hence, pilot data are a requisite for MultiML.)
- MultiPower is not directly applicable to ML sample classification because significance threshold and effect size do not apply; only the ER-based MultiML route is valid here
- Estimates depend on the quality of pilot data: inputs must be properly preprocessed and batch effects removed, since the tool performs no such handling
- Power considerations differ from differential-analysis power: power diminishes as the number of features increases due to multiple testing corrections, so PSS trends should not be conflated with MultiPower sample-size results

## Evidence

- [abstract] MultiML estimates sample size for machine learning classification problems based on multi-omic data: "MultiML, an algorithm to estimate sample size for machine learning classification problems based on multi-omic data"
- [results] Random forest PSS decreases with more combined omics at ER 0.01: [evidence span withheld — anchor status: not_found]
- [discussion] MultiML cannot run without pilot data: "Hence, pilot data are a requisite for MultiML."
- [discussion] MultiPower not applicable to ML classification: "The MultiPower approach is not directly applicable to ML methods used for sample classification, as in this case the basic parameters of the power calculation—significance threshold and effect"
- [readme] MultiML usage instruction from README: "designed to help users determine the optimal sample size required to control for classification error rates when using one or more omics datasets"
