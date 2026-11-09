---
name: multi-omics-sample-size-estimation
description: 'Use when designing a multi-omic experiment and you must size the cohort:
  MultiPower for differential-feature power across omics, or MultiML when a classifier
  such as random forest must hit a target classification error rate (e.g., ER=0.01)
  on combined omic platforms.'
when_to_use_negative:
- 'You are sizing an ML sample-classification study but reach for MultiPower: its
  power calculation needs a significance threshold and effect size, which do not apply
  to classification error rates — use MultiML instead.'
- 'No pilot data exist for the study system: MultiML cannot run without prior data,
  and MultiPower parameter estimates would have to be set manually.'
- 'The omic data are raw or still contain batch effects: the tools assume inputs are
  already preprocessed with batch effects removed and do no handling themselves.'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0202
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: MultiPower
  role: R package performing statistical power studies for multi-omics experiments;
    used to estimate and assess optimal sample size for differential-feature analysis
    across omics via MultiPower/MultiGroupPower/optimalRep/postMultiPower/powerPlot
  repo: github.com/ConesaLab/MultiPower
- name: MultiML
  role: Complementary algorithm in the same package that estimates the optimal sample
    size required by an ML classifier (random forest in the surfaced task) to achieve
    a target classification ER from pilot multi-omic data
  repo: github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:849d485f3003ead6296524d57fe3086102bb078d78a5eab51a61da245cbc2973
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiML
  - MultiPower
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0202
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# multi-omics-sample-size-estimation

## Summary

Estimates the optimal number of samples for multi-omic experiments: MultiPower performs power calculations for differential-feature analysis across omic platforms, while MultiML predicts the sample size an ML classifier (e.g., random forest) needs to reach a target classification error rate (e.g., ER=0.01) learned from pilot multi-omic data such as TCGA Glioblastoma feature matrices.

## When to use

Use when designing a multi-omic experiment and you must size the cohort: MultiPower for differential-feature power across omics, or MultiML when a classifier such as random forest must hit a target classification error rate (e.g., ER=0.01) on combined omic platforms.

## When NOT to use

- You are sizing an ML sample-classification study but reach for MultiPower: its power calculation needs a significance threshold and effect size, which do not apply to classification error rates — use MultiML instead.
- No pilot data exist for the study system: MultiML cannot run without prior data, and MultiPower parameter estimates would have to be set manually.
- The omic data are raw or still contain batch effects: the tools assume inputs are already preprocessed with batch effects removed and do no handling themselves.

## Inputs

- Preprocessed, batch-effect-free pilot multi-omic feature matrices per platform (e.g., TCGA Glioblastoma mRNA, miRNA, methylation, proteomics, clinical/copy-number), covering two to five platforms
- Target classification error rate for MultiML (e.g., ER=0.01) and the intended ML classifier (e.g., random forest)
- For MultiPower: pilot omic data with design arguments (data, groups, type, omicPower, averagePower, fdr, cost, equalSize, max.size)

## Outputs

- Predicted sample sizes (PSS) per omic-platform combination at the target ER, e.g. multiml_gbm_sample_sizes.csv
- Optimal sample-size estimates with graphical outputs (powerPlots, summaryPlot) for experimental design decision-making
- Per-omic power and average power across omics from MultiPower for differential-feature studies

## How to apply

Install the MultiPower R package from GitHub with devtools::install_github('ConesaLab/MultiPower') after adding its dependencies FDRsampsize and lpSolve. For classification-based sizing, load preprocessed, batch-effect-free pilot multi-omic feature matrices (e.g., TCGA Glioblastoma mRNA, miRNA, methylation, proteomics, clinical/copy-number platforms) and configure MultiML with the intended classifier — random forest in the surfaced task — and the target classification error rate (ER=0.01). Run the estimation separately for each combination of two, three, four and five omic platforms, letting MultiML learn the predicted sample size (PSS) from the pilot data, since pilot data are a requisite and the tool performs no preprocessing or batch correction. Collect the per-combination PSS values into one table (e.g., multiml_gbm_sample_sizes.csv) and verify they are monotonically non-increasing as platforms are added, reflecting complementary multi-omic information. For differential-feature sizing rather than ML classification, use the MultiPower workflow (MultiGroupPower/MultiPower/optimalRep/postMultiPower/powerPlot) with parameters omicPower, averagePower, fdr, cost, equalSize and max.size, because MultiPower's significance-threshold/effect-size framework does not transfer to classification error rates.

## Related tools

- **MultiPower** (R package performing statistical power studies for multi-omics experiments; used to estimate and assess optimal sample size for differential-feature analysis across omics via MultiPower/MultiGroupPower/optimalRep/postMultiPower/powerPlot) — github.com/ConesaLab/MultiPower
- **MultiML** (Complementary algorithm in the same package that estimates the optimal sample size required by an ML classifier (random forest in the surfaced task) to achieve a target classification ER from pilot multi-omic data) — github.com/ConesaLab/MultiPower

## Examples

```
install.packages("devtools"); devtools::install_github("ConesaLab/MultiPower")
```

## Evaluation signals

- The output table contains exactly one PSS value per omic-platform combination (two through five platforms) for the ER=0.01 target.
- PSS values are monotonically non-increasing as more omic platforms are combined, matching the reported Fig. 5b-d behaviour.
- MultiML was run on pilot data that were preprocessed and batch-effect free, since the tool performs no preprocessing itself.
- The MultiPower package installed cleanly from GitHub with the FDRsampsize and lpSolve dependencies present.
- The classifier and error rate recorded with the sample-size table match the configured settings (random forest, ER=0.01).

## Limitations

- MultiML requires pilot data; it cannot run without prior data.
- MultiPower parameter estimates should be calculated from pilot data; otherwise users must provide them manually.
- Input data must be properly preprocessed with batch effects removed; the tools do not handle preprocessing or batch correction.
- MultiPower is not directly applicable to ML methods for sample classification because significance threshold and effect size are not applicable parameters.

## Evidence

- [abstract] MultiPower is complemented with MultiML, an algorithm to estimate sample size for machine learning classification problems based on multi-omic data: "MultiPower is complemented with MultiML, an algorithm to estimate sample size for machine learning classification problems based on multi-omic data"
- [abstract] we formulate the MultiPower method to estimate and assess the optimal sample size in a multi-omics experiment: "we formulate the MultiPower method to estimate and assess the optimal sample size in a multi-omics experiment"
- [intro] we present MultiML, an R method to obtain the optimal sample size required by ML approaches to achieve a target classification ER: "we present MultiML, an R method to obtain the optimal sample size required by ML approaches to achieve a target classification ER"
- [methods] help users determine the optimal sample size required to control for classification error rates when using one or more omics datasets: "help users determine the optimal sample size required to control for classification error rates when using one or more omics datasets"
- [methods] devtools::install_github(“ConesaLab/MultiPower”): "devtools::install_github(“ConesaLab/MultiPower”)"
- [discussion] The MultiML strategy calculates samples size for multi-omic applications, where the classification ER can be used as a measure of performance.: "The MultiML strategy calculates samples size for multi-omic applications, where the classification ER can be used as a measure of performance."
- [discussion] Hence, pilot data are a requisite for MultiML.: "Hence, pilot data are a requisite for MultiML."
- [discussion] Data should have been properly preprocessed and eventual batch effects, removed.: "Data should have been properly preprocessed and eventual batch effects, removed."
- [discussion] The MultiPower approach is not directly applicable to ML methods used for sample classification: "The MultiPower approach is not directly applicable to ML methods used for sample classification"
- [readme] The MultiML method is included as a complementary tool to MultiPower,: "The MultiML method is included as a complementary tool to MultiPower,"
