---
name: iterative-sample-size-convergence-testing
description: Use when estimating the optimal sample size for an ML classification
  task on multi-omic data, and you need to determine how many sampling ticks suffice
  before fitting a learning-curve model to tick-vs-ER data.
when_to_use_negative:
- MultiPower-style power calculation for differential feature analysis is intended
  instead — significance threshold and effect size are not applicable to ML classification
  problems
- No pilot data are available — pilot data are a requisite for MultiML
- Input data are not preprocessed and batch-effect free — the tool does not handle
  batch effects internally
tools:
- name: MultiML
  role: Provides the iterative tick-append loop with the 12% relative-change stopping
    criterion and the downstream P-spline learning-curve fit for sample-size prediction
  repo: https://github.com/ConesaLab/MultiPower/blob/master/MultiPower_UsersGuide.pdf
- name: MultiPower
  role: R package repository that hosts MultiML and its user guide
  repo: github.com/ConesaLab/MultiPower
- name: R
  role: Implementation language for MultiML and its P-spline regression fitting
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:f6b52cabbea493516466d67bb85f1bea49bd72015838ba3854399cec288347ef
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiML
  - MultiPower
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# iterative-sample-size-convergence-testing

## Summary

Iteratively appends new sample-size ticks and refits the average ML classification error rate (ER) at each tick until the relative change between successive ticks falls below a 12% stopping threshold, providing the stable tick-vs-ER data on which MultiML's P-spline learning curve is subsequently fitted.

## When to use

Use when estimating the optimal sample size for an ML classification task on multi-omic data, and you need to determine how many sampling ticks suffice before fitting a learning-curve model to tick-vs-ER data.

## When NOT to use

- MultiPower-style power calculation for differential feature analysis is intended instead — significance threshold and effect size are not applicable to ML classification problems
- No pilot data are available — pilot data are a requisite for MultiML
- Input data are not preprocessed and batch-effect free — the tool does not handle batch effects internally

## Inputs

- Per-tick average classification error rate (ER) table linking each sampling tick to its average ML classification ER
- Pilot multi-omic classification data used to refit the average ER at each new tick

## Outputs

- Converged tick-vs-ER sequence (n1..nt vs ER1..ERt) suitable for the P-spline learning-curve fit
- multiml_pspline_sample_size.csv containing fitted curve coordinates and the predicted sample size for the target ER

## How to apply

1) Load the per-tick average classification error rate (ER) table linking each sampling tick to its average ML classification ER. 2) Iteratively append new sample-size ticks, refitting the average ER at each tick. 3) Stop when the relative change in average ER between successive ticks falls below the 12% stopping threshold. 4) Pass the converged tick-vs-ER data to the P-spline fit (first-order difference penalty, df = number of ticks - 1) to obtain the learning curve used to predict classification ERs for sample sizes larger than Nmax. The rationale is that fitting a smoothed curve to a non-converged sequence would propagate unstable ER estimates into the extrapolated sample-size prediction.

## Related tools

- **MultiML** (Provides the iterative tick-append loop with the 12% relative-change stopping criterion and the downstream P-spline learning-curve fit for sample-size prediction) — https://github.com/ConesaLab/MultiPower/blob/master/MultiPower_UsersGuide.pdf
- **MultiPower** (R package repository that hosts MultiML and its user guide) — github.com/ConesaLab/MultiPower
- **R** (Implementation language for MultiML and its P-spline regression fitting)

## Examples

```
install.packages("devtools"); devtools::install_github("ConesaLab/MultiPower")
```

## Evaluation signals

- The iterative procedure converged under the 12% relative-change stopping criterion (log the relative change at each tick and confirm the final successive change is below threshold)
- The fitted P-spline (df = ticks - 1, first-order difference penalty) yields a monotone, smoothed ER-vs-sample-size curve
- A single target-ER sample size is reported in multiml_pspline_sample_size.csv
- Extrapolation of the fitted curve is only applied for sample sizes larger than Nmax
- The number of fitted spline degrees of freedom equals the number of ticks minus 1

## Limitations

- MultiML requires pilot data; it cannot run without prior data
- The stopping criterion is a 12% relative change in average ER between successive ticks, so very noisy ER estimates at small ticks can delay or prevent convergence
- Data must be preprocessed and batch-effect free before applying the method; no handling within the tool

## Evidence

- [methods] Iterative tick-append stopping rule: [evidence span withheld — anchor status: not_found]
- [methods] P-spline fit on converged tick data: [evidence span withheld — anchor status: not_found]
- [intro] MultiML purpose: "we present MultiML, an R method to obtain the optimal sample size required by ML approaches to achieve a target classification ER"
- [discussion] Pilot data requirement: "Hence, pilot data are a requisite for MultiML."
- [readme] README description of MultiML role: "designed to help users determine the optimal sample size required to control for classification error rates when using one or more omics datasets"
