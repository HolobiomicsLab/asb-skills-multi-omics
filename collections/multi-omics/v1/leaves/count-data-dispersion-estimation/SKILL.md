---
name: count-data-dispersion-estimation
description: Use when pilot multi-omic count matrices (preprocessed, batch-effect-free)
  are available and per-feature dispersion/variability parameters must be estimated
  to feed downstream sample-size/power estimation (e.g., optimalRep in MultiPower).
when_to_use_negative:
- Input data have not been properly preprocessed or still contain batch effects —
  MultiPower provides no handling for these within the tool
- No pilot data are available — MultiPower parameter estimates depend on pilot data,
  otherwise parameters must be set manually
- The goal is sample-size estimation for machine learning classification (use MultiML
  instead), since significance threshold and effect size are not applicable in that
  setting
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_2269
tools:
- name: MultiPower
  role: R package whose CohenFilter(data, d, parameters) function computes Cohen's
    d, flags the pseudo-DE set M1, and derives pooled-SD/variance, mean counts, dispersion
    and fold-change percentile parameters for downstream power estimation via optimalRep
  repo: github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:15e611c4d9620c2eacf6b23b5759019d1b5907f63171e11ff4f47f11704341ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/MultiPower
  tools:
  - MultiPower
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Count-data dispersion estimation

## Summary

Derives the variability/dispersion input required for statistical power calculations from pilot multi-omic count data, by computing per-feature pooled standard deviation (PSD) over pseudo-DE features flagged with Cohen's d and summarizing them as user-chosen percentiles (Pk and P100-k) as implemented in MultiPower's CohenFilter.

## When to use

Use when pilot multi-omic count matrices (preprocessed, batch-effect-free) are available and per-feature dispersion/variability parameters must be estimated to feed downstream sample-size/power estimation (e.g., optimalRep in MultiPower).

## When NOT to use

- Input data have not been properly preprocessed or still contain batch effects — MultiPower provides no handling for these within the tool
- No pilot data are available — MultiPower parameter estimates depend on pilot data, otherwise parameters must be set manually
- The goal is sample-size estimation for machine learning classification (use MultiML instead), since significance threshold and effect size are not applicable in that setting

## Inputs

- Preprocessed, batch-effect-free pilot count matrix per omic data type (R data object)
- Cohen's d threshold d0 for defining the pseudo-DE feature set M1
- User-chosen percentile k of the per-feature pooled-standard-deviation distribution

## Outputs

- Parameter object with SD, variance, mean counts, dispersion, and fold-change percentiles (Pk and P100-k) over pseudo-DE features, suitable for optimalRep(parameters, ...)
- Parameter table (e.g., cohens_d_parameters.csv)

## How to apply

Load the preprocessed, batch-effect-free pilot count matrix for each omic data type into R. Call the exported MultiPower function CohenFilter(data, d, parameters) to compute Cohen's d for all features and define the pseudo-DE set M1 as features with d greater than the supplied d0 threshold. Over the M1 features only, compute the pooled standard deviation and variance, mean counts, dispersion, and fold-change per omic; the variability input to power calculations is the pooled standard deviation computed per feature. Summarize each parameter as the Pk and P100-k percentiles over the pseudo-DE features, so the user-selected percentile of the per-feature PSD distribution becomes the dispersion parameter for power estimation. Write the result (e.g., cohens_d_parameters.csv) and verify that every parameter field — SD, variance, mean counts, dispersion, and fold-change percentiles — is present and finite before passing the parameters object to optimalRep(parameters, ...).

## Related tools

- **MultiPower** (R package whose CohenFilter(data, d, parameters) function computes Cohen's d, flags the pseudo-DE set M1, and derives pooled-SD/variance, mean counts, dispersion and fold-change percentile parameters for downstream power estimation via optimalRep) — github.com/ConesaLab/MultiPower

## Examples

```
install.packages("devtools"); devtools::install_github("ConesaLab/MultiPower"); parameters <- CohenFilter(data, d, parameters)
```

## Evaluation signals

- Every parameter field in the output (SD, variance, mean counts, dispersion, fold-change percentiles) is present and finite
- Pseudo-DE set M1 contains only features with Cohen's d greater than the supplied d0 threshold
- Percentile summaries correspond to the user-specified Pk and P100-k percentiles over the M1 features
- The parameters object is accepted by optimalRep(parameters, ...) without missing-argument errors
- Input count matrices were verified as preprocessed and batch-effect-free before running CohenFilter

## Limitations

- Estimates for MultiPower parameters are optimally calculated from pilot data; without pilot data users must set parameters manually
- Input data must be preprocessed and batch effects removed beforehand; the tool does not handle this internally
- Power diminishes as the number of features increases due to multiple testing corrections controlling false positives

## Evidence

- [methods] CohenFilter derives pseudo-DE set and dispersion percentiles: [evidence span withheld — anchor status: not_found]
- [discussion] Estimates from pilot data or manual: "Estimates for MultiPower parameters are optimally calculated from pilot data, although they can also be manually provided."
- [discussion] Preprocessing requirement: "Data should have been properly preprocessed and eventual batch effects, removed."
- [readme] Installation from GitHub: "devtools::install_github(“ConesaLab/MultiPower”)"
