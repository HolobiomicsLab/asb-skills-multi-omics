---
name: penalized-pspline-regression
description: Use when estimating the sample size a machine learning classifier needs
  on multi-omic data to reach a user-specified target classification error rate, from
  an ER-vs-sample-size table.
when_to_use_negative:
- The goal is differential-features power/sample-size calculation — MultiPower's significance-threshold/effect-size
  framework applies there, and the article states MultiPower 'is not directly applicable
  to ML methods used for sample classification', while this P-spline fit is the MultiML
  counterpart
- No pilot multi-omic data exist — the article notes 'pilot data are a requisite for
  MultiML'
- The ER-vs-tick series did not converge under the 12% stopping criterion, so the
  tick-vs-ER points are not a reliable basis for fitting
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MultiML
  role: Algorithm embedding the P-spline learning-curve fit to estimate optimal sample
    size for ML classification on multi-omic data
  repo: https://github.com/ConesaLab/MultiPower
- name: MultiPower
  role: Companion R package distributed on GitHub that contains MultiML scripts and
    user guide
  repo: github.com/ConesaLab/MultiPower
- name: R
  role: Language in which the penalized P-spline regression and MultiML procedure
    are implemented
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:029717c4974d965a2d78caebebcd757dcbc9c16531114b5906779d7006096000
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/MultiPower
  tools:
  - MultiML
  - MultiPower
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# penalized-pspline-regression

## Summary

Fits a first-order-smoothed penalized P-spline regression to per-tick sample sizes (n1–nt) versus average ML classification error rates (ER1–ERt) within the MultiML algorithm, yielding a smoothed learning curve that is inverted to predict the optimal sample size for a target classification ER.

## When to use

Use when estimating the sample size a machine learning classifier needs on multi-omic data to reach a user-specified target classification error rate, from an ER-vs-sample-size table.

## When NOT to use

- The goal is differential-features power/sample-size calculation — MultiPower's significance-threshold/effect-size framework applies there, and the article states MultiPower 'is not directly applicable to ML methods used for sample classification', while this P-spline fit is the MultiML counterpart
- No pilot multi-omic data exist — the article notes 'pilot data are a requisite for MultiML'
- The ER-vs-tick series did not converge under the 12% stopping criterion, so the tick-vs-ER points are not a reliable basis for fitting

## Inputs

- Table of sampling ticks versus average classification error rates (ER1–ERt), converged under the 12% relative-change stopping criterion
- User-specified target classification error rate
- Multi-omic pilot data used to train the per-tick classifiers

## Outputs

- Fitted P-spline learning-curve coordinates (sample size vs. smoothed ER)
- Predicted optimal sample size for the target classification error rate
- multiml_pspline_sample_size.csv

## How to apply

Load the per-tick average classification error rate table linking each sampling tick to its average ML classification ER, having appended ticks iteratively until the relative change in average ER between successive ticks fell below the 12% stopping threshold. Fit a penalized P-spline regression to the tick-vs-ER data using a first-order difference penalty on the spline coefficients, setting the degrees of freedom equal to the number of ticks minus 1. Rationale: smoothing removes tick-to-tick noise in the ER estimates while the first-order penalty enforces a stable, approximately monotone learning curve. Invert the fitted curve to predict the sample size that achieves the user-specified target ER, including extrapolation to sample sizes larger than Nmax. Write the fitted curve coordinates and the predicted sample size to multiml_pspline_sample_size.csv.

## Related tools

- **MultiML** (Algorithm embedding the P-spline learning-curve fit to estimate optimal sample size for ML classification on multi-omic data) — https://github.com/ConesaLab/MultiPower
- **MultiPower** (Companion R package distributed on GitHub that contains MultiML scripts and user guide) — github.com/ConesaLab/MultiPower
- **R** (Language in which the penalized P-spline regression and MultiML procedure are implemented)

## Examples

```
install.packages("devtools"); devtools::install_github("ConesaLab/MultiPower")  # then run MultiML scripts to fit the P-spline and write multiml_pspline_sample_size.csv
```

## Evaluation signals

- The iterative tick-appending phase converged: the relative change in average ER between successive ticks fell below the 12% stopping threshold
- The fitted P-spline uses df = (number of ticks - 1) and produces a monotone, smoothed ER-vs-sample-size curve
- A single, finite sample size is successfully predicted for the user-specified target classification ER (including extrapolation beyond Nmax)
- The fitted curve coordinates and predicted sample size are written to multiml_pspline_sample_size.csv in the expected format
- Predicted ER at each observed tick reasonably matches the average classification ER from the input table (smoothed, not overfit)

## Limitations

- MultiML requires pilot data; it cannot run without prior data
- Input data must be properly preprocessed and batch effects removed before applying the MultiPower/MultiML framework
- The P-spline prediction relies on extrapolating the fitted curve for sample sizes larger than Nmax, so target ERs far below the observed range are less reliable

## Evidence

- [methods] Fit a penalized P-spline regression to the tick-vs-ER data using a first-order difference penalty on the spline coefficients: [evidence span withheld — anchor status: not_found]
- [abstract] MultiML is an algorithm to estimate sample size for ML classification on multi-omic data: "MultiPower is complemented with MultiML, an algorithm to estimate sample size for machine learning classification problems based on multi-omic data"
- [intro] MultiML is an R method to obtain the optimal sample size required by ML approaches to achieve a target classification ER: "we present MultiML, an R method to obtain the optimal sample size required by ML approaches to achieve a target classification ER"
- [readme] MultiML installation is via the MultiPower R package from GitHub: "devtools::install_github(“ConesaLab/MultiPower”)"
