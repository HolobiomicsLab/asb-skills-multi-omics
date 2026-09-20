---
name: machine-learning-sample-size-estimation
description: Use when designing a multi-omic ML classification study and you need
  the number of samples required to control classification ER, given pilot multi-omic
  data with per-tick average classification error rates.
when_to_use_negative:
- The goal is differential features analysis power (significance threshold and effect
  size apply) — use MultiPower instead, since MultiPower is not directly applicable
  to ML sample classification
- No pilot data are available — MultiML cannot run without prior data
- Input data are not preprocessed or still contain batch effects — MultiML does not
  handle preprocessing or batch-effect removal
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_2269
tools:
- name: MultiML
  role: R method that fits the P-spline learning curve to tick-vs-ER data and inverts
    it to predict the optimal sample size for a target classification ER
  repo: https://github.com/ConesaLab/MultiPower
- name: MultiPower
  role: Companion R package containing MultiML and its dependencies; installed from
    GitHub and used alongside the MultiML User Guide
  repo: https://github.com/ConesaLab/MultiPower
- name: R
  role: Runtime environment; MultiML is an R method and its dependencies are installed
    via install.packages() and devtools::install_github()
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:2eaa56a2c8eb2562e53baa65b5590919abe2e51f66818c92350d4f22d68edbdd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/MultiPower
  tools:
  - MultiML
  - MultiPower
  - R
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# machine-learning-sample-size-estimation

## Summary

Estimates the optimal sample size required by machine learning classification approaches to reach a user-specified target classification error rate (ER) on multi-omic data, by fitting a first-order-smoothed penalized P-spline learning curve to per-tick sample-size vs average-ER points and inverting it (MultiML).

## When to use

Use when designing a multi-omic ML classification study and you need the number of samples required to control classification ER, given pilot multi-omic data with per-tick average classification error rates.

## When NOT to use

- The goal is differential features analysis power (significance threshold and effect size apply) — use MultiPower instead, since MultiPower is not directly applicable to ML sample classification
- No pilot data are available — MultiML cannot run without prior data
- Input data are not preprocessed or still contain batch effects — MultiML does not handle preprocessing or batch-effect removal

## Inputs

- Per-tick average classification error rate (ER) table linking each sampling tick (n1..nt) to its average ML classification ER (ER1..ERt)
- Pilot multi-omic data used to train/evaluate classifiers at successive sample-size ticks
- User-specified target classification error rate

## Outputs

- multiml_pspline_sample_size.csv containing fitted P-spline curve coordinates and the predicted optimal sample size for the target ER

## How to apply

Load the per-tick table linking each sampling tick (n1..nt) to its average ML classification ER (ER1..ERt); iteratively append new sample-size ticks and refit the average ER at each tick until the relative change in average ER between successive ticks falls below the 12% stopping threshold. Fit a penalized P-spline regression to the tick-vs-ER data using a first-order difference penalty on the spline coefficients, with degrees of freedom set to (number of ticks - 1), yielding a monotone, smoothed learning curve. Invert the fitted curve to predict the sample size achieving the user's target ER, and extrapolate ERs for sample sizes larger than Nmax. Write the fitted curve coordinates and predicted sample size to multiml_pspline_sample_size.csv.

## Related tools

- **MultiML** (R method that fits the P-spline learning curve to tick-vs-ER data and inverts it to predict the optimal sample size for a target classification ER) — https://github.com/ConesaLab/MultiPower
- **MultiPower** (Companion R package containing MultiML and its dependencies; installed from GitHub and used alongside the MultiML User Guide) — https://github.com/ConesaLab/MultiPower
- **R** (Runtime environment; MultiML is an R method and its dependencies are installed via install.packages() and devtools::install_github())

## Examples

```
devtools::install_github("ConesaLab/MultiPower")
```

## Evaluation signals

- The iterative tick-append procedure converged under the 12% relative-change stopping criterion on average ER between successive ticks
- The fitted P-spline (df = number of ticks - 1, first-order difference penalty) yields a monotone, smoothed ER-vs-sample-size curve
- A single, finite predicted sample size for the user-specified target ER is reported in multiml_pspline_sample_size.csv
- The output CSV contains both the fitted curve coordinates and the predicted sample size
- Predicted ERs for sample sizes larger than Nmax come from extrapolation of the fitted P-spline curve, not raw tick averages

## Limitations

- MultiML requires pilot data; it cannot run without prior data
- Predictions for sample sizes larger than Nmax rely on extrapolation of the fitted P-spline learning curve
- MultiPower (and by extension the framework) is not directly applicable to ML sample classification because significance threshold and effect size are not applicable; hence the separate MultiML learning-curve approach is required
- Input data must be preprocessed and batch-effect free before applying the method; no handling within the tool

## Evidence

- [abstract] MultiML estimates sample size: "MultiML, an algorithm to estimate sample size for machine learning classification problems based on multi-omic data"
- [intro] MultiML definition: "we present MultiML, an R method to obtain the optimal sample size required by ML approaches to achieve a target classification ER"
- [full_text] P-spline fit detail: [evidence span withheld — anchor status: not_found]
- [full_text] Workflow validation detail: [evidence span withheld — anchor status: not_found]
- [readme] MultiML usage guidance: "The MultiML method is included as a complementary tool to MultiPower, and is designed to help users determine the optimal sample size required to control for classification error rates"
- [discussion] Pilot data requirement: "Hence, pilot data are a requisite for MultiML."
