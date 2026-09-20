---
name: effect-size-estimation-cohens-d
description: Use when a pilot multi-omic count matrix is available and MultiPower's
  optimalRep/MultiPower functions require input parameters (variability, effect size,
  dispersion percentiles) derived from per-feature effect sizes.
when_to_use_negative:
- Input data have not been properly preprocessed or batch effects have not been removed
  — MultiPower requires pilot data that are already clean, with no handling within
  the tool
- No pilot data are available and parameters cannot be estimated — users would have
  to provide parameters manually instead
- The goal is machine-learning sample classification — MultiPower's significance threshold
  and effect size parameters are not applicable there; MultiML is the appropriate
  method
edam_operation: http://edamontology.org/operation_3231
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3520
tools:
- name: MultiPower
  role: R package whose CohenFilter function computes Cohen's d per feature and whose
    power-calculation framework (optimalRep, MultiPower) consumes the derived PSD-percentile
    parameters
  repo: github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:b92160f2e3814c919891f739195d3455de9a57d7418c7ea7a08fdf81ccab55c9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiPower
  edam_operation: http://edamontology.org/operation_3231
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# effect-size-estimation-cohens-d

## Summary

Computes Cohen's d effect sizes per feature from a preprocessed multi-omic pilot count matrix using MultiPower's CohenFilter, flags the pseudo-DE feature set M1 (d > d0), and derives the pooled-standard-deviation percentile parameters MultiPower needs for downstream power/sample-size estimation.

## When to use

Use when a pilot multi-omic count matrix is available and MultiPower's optimalRep/MultiPower functions require input parameters (variability, effect size, dispersion percentiles) derived from per-feature effect sizes.

## When NOT to use

- Input data have not been properly preprocessed or batch effects have not been removed — MultiPower requires pilot data that are already clean, with no handling within the tool
- No pilot data are available and parameters cannot be estimated — users would have to provide parameters manually instead
- The goal is machine-learning sample classification — MultiPower's significance threshold and effect size parameters are not applicable there; MultiML is the appropriate method

## Inputs

- Preprocessed, batch-effect-free pilot count matrix per omic data type
- Cohen's d effect-size threshold d0 (supplied via CohenFilter's d/parameters arguments)

## Outputs

- Pseudo-DE feature set M1 (features with d > d0)
- Per-omic pooled standard deviation and variance over M1 features
- Per-omic mean counts, dispersion, and fold-change over M1 features
- Parameter object of Pk and P100-k percentiles for downstream optimalRep()
- cohens_d_parameters.csv parameter table

## How to apply

Load the preprocessed, batch-effect-free pilot count matrix for each omic data type into R. Call the exported CohenFilter(data, d, parameters) function to compute Cohen's d for all features and flag the pseudo-DE set M1 as features whose d exceeds the supplied d0 threshold. Restricting to M1 features, compute the pooled standard deviation and variance, mean counts, dispersion, and fold-change per omic. Summarize each parameter as the Pk and P100-k percentiles over the pseudo-DE features, yielding the parameters object expected by optimalRep(parameters, ...). The rationale is that MultiPower derives variability from the per-feature pooled standard deviation, and the user selects a percentile of the per-feature PSD distribution as the dispersion parameter for power estimation; percentile outputs (SD, variance, mean counts, dispersion, fold-change) must all be present and finite before proceeding.

## Related tools

- **MultiPower** (R package whose CohenFilter function computes Cohen's d per feature and whose power-calculation framework (optimalRep, MultiPower) consumes the derived PSD-percentile parameters) — github.com/ConesaLab/MultiPower

## Examples

```
install.packages("devtools"); devtools::install_github("ConesaLab/MultiPower"); parameters <- CohenFilter(data, d, parameters)
```

## Evaluation signals

- cohens_d_parameters.csv contains every parameter field (SD, variance, mean counts, dispersion, fold-change percentiles) and all values are finite
- M1 contains only features whose Cohen's d exceeds the supplied d0 threshold
- Percentile summaries are reported at exactly the Pk and P100-k percentiles over the pseudo-DE features
- The produced parameters object is accepted by optimalRep(parameters, ...) without missing-argument errors
- Input matrices were loaded per omic after batch-effect removal (preprocessing check passed before CohenFilter)

## Limitations

- MultiPower parameter estimates are optimally calculated from pilot data, although they can also be manually provided; without pilot data this skill cannot run
- Power diminishes as the number of features increases due to multiple testing corrections controlling false positives
- The MultiPower approach is not directly applicable to ML methods for sample classification, as significance threshold and effect size are not applicable in that setting
- Input must be preprocessed and batch-effect free; MultiPower performs no such handling internally

## Evidence

- [other] MultiPower derives its variability input from the pooled standard deviation computed per feature, and users set a percentile of the per-feature PSD distribution: [evidence span withheld — anchor status: not_found]
- [methods] CohenFilter export evidence: "- `CohenFilter(data, d, parameters)`"
- [methods] optimalRep consumption of parameters object: "- `optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect)`"
- [discussion] pilot-data prerequisite: "Estimates for MultiPower parameters are optimally calculated from pilot data, although they can also be manually provided."
- [discussion] preprocessing prerequisite: "Data should have been properly preprocessed and eventual batch effects, removed."
