---
name: classification-error-rate-curve-smoothing
description: Use when pilot multi-omic data have been iteratively resampled (n1 to
  nt) with a ML classifier, average ER per tick is tabulated, and the goal is a smoothed
  ER-vs-sample-size learning curve to estimate the optimal sample size for a target
  ER.
when_to_use_negative:
- The goal is sample size for differential-feature analysis (use MultiPower's power
  calculation instead — significance threshold and effect size are not applicable
  to ML classification)
- No pilot multi-omic data are available — MultiML requires pilot data to build the
  ER curve
- The iterative tick procedure has not converged under the 12% stopping threshold,
  so the ER table is incomplete and the spline fit would be unreliable
edam_operation: http://edamontology.org/operation_3726
tools:
- name: MultiML
  role: Implements the iterative resampling and P-spline learning-curve fit used to
    estimate the optimal sample size for a target classification ER in multi-omic
    ML classification
  repo: github.com/ConesaLab/MultiPower
- name: MultiPower
  role: Companion R package that ships MultiML; installed via devtools from GitHub
    and documented in the MultiML User Guide
  repo: github.com/ConesaLab/MultiPower
- name: R
  role: Language in which the P-spline regression and MultiML workflow are implemented
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:9d839cba8b1201a7f5669aa77285d992ef0d32323f5b1e92658c91432e5b935b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiML
  - MultiPower
  - R
  edam_operation: http://edamontology.org/operation_3726
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# classification-error-rate-curve-smoothing

## Summary

Fits a first-order-smoothed penalized P-spline regression to per-tick sample sizes and average classification error rates (ER1–ERt) from the MultiML iterative sampling procedure, yielding a smooth learning curve used to predict the sample size needed for a target classification ER in multi-omic ML classification.

## When to use

Use when pilot multi-omic data have been iteratively resampled (n1 to nt) with a ML classifier, average ER per tick is tabulated, and the goal is a smoothed ER-vs-sample-size learning curve to estimate the optimal sample size for a target ER.

## When NOT to use

- The goal is sample size for differential-feature analysis (use MultiPower's power calculation instead — significance threshold and effect size are not applicable to ML classification)
- No pilot multi-omic data are available — MultiML requires pilot data to build the ER curve
- The iterative tick procedure has not converged under the 12% stopping threshold, so the ER table is incomplete and the spline fit would be unreliable

## Inputs

- Per-tick average classification error rate table (sampling ticks n1..nt vs average ER1..ERt) from MultiML iterative resampling of one or more omics datasets
- User-specified target classification error rate

## Outputs

- Fitted P-spline learning-curve coordinates (smoothed ER vs sample size)
- Predicted optimal sample size for the target classification ER, written to multiml_pspline_sample_size.csv

## How to apply

Load the table linking each sampling tick to its average ML classification ER, after the iterative tick-append step has stopped (relative change in average ER between successive ticks < 12%). Fit a penalized P-spline regression to the tick-vs-ER points using a first-order difference penalty on the spline coefficients, setting the degrees of freedom equal to (number of ticks - 1); the penalty enforces a monotone, smoothed learning curve rather than interpolating noisy per-tick ER estimates. Invert the fitted P-spline curve to solve for the sample size that achieves the user-specified target classification ER. Write the fitted curve coordinates and the predicted sample size to multiml_pspline_sample_size.csv. This smoothing step is what lets MultiML extrapolate ER predictions beyond Nmax, the largest resampled tick.

## Related tools

- **MultiML** (Implements the iterative resampling and P-spline learning-curve fit used to estimate the optimal sample size for a target classification ER in multi-omic ML classification) — github.com/ConesaLab/MultiPower
- **MultiPower** (Companion R package that ships MultiML; installed via devtools from GitHub and documented in the MultiML User Guide) — github.com/ConesaLab/MultiPower
- **R** (Language in which the P-spline regression and MultiML workflow are implemented)

## Examples

```
install.packages("devtools"); devtools::install_github("ConesaLab/MultiPower")
```

## Evaluation signals

- The iterative tick procedure converged under the 12% relative-change stopping criterion before the spline was fitted
- The fitted P-spline (df = ticks - 1, first-order difference penalty) yields a monotone, smoothed ER-vs-sample-size curve with no non-physical upturns
- Inverting the fitted curve returns a single sample-size value for the user-specified target ER, not multiple or zero solutions
- The fitted curve coordinates plus predicted sample size are present in multiml_pspline_sample_size.csv with a complete row per output point
- Predicted sample size is at least Nmax (the largest resampled tick) whenever the target ER is smaller than the observed ER at Nmax

## Limitations

- MultiML requires pilot data — it cannot run without prior data to build the ER curve
- The ER table must come from converged iterative resampling; stopping before the 12% threshold is met invalidates the extrapolation
- MultiML is specific to ML sample classification; MultiPower's power parameters (significance threshold, effect size) do not transfer to this skill

## Evidence

- [methods] P-spline rationale: [evidence span withheld — anchor status: not_found]
- [methods] extrapolation role: [evidence span withheld — anchor status: not_found]
- [readme] README MultiML purpose: "designed to help users determine the optimal sample size required to control for classification error rates when using one or more omics datasets."
- [discussion] pilot data requirement: "Hence, pilot data are a requisite for MultiML."
