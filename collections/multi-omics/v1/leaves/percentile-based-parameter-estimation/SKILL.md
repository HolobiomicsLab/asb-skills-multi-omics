---
name: percentile-based-parameter-estimation
description: 'Use when designing a multi-omic experiment and you need MultiPower input
  parameters estimated from pilot data: a count matrix per omic plus a Cohen''s d
  threshold d0 are available, and sample size will be computed via optimalRep(parameters,
  ...).'
when_to_use_negative:
- Input data are not yet preprocessed or still contain batch effects — MultiPower
  requires that data 'should have been properly preprocessed and eventual batch effects,
  removed' and offers no handling within the tool.
- The goal is sample-size estimation for machine learning classification — MultiPower's
  significance threshold and effect size parameters are not applicable there; use
  MultiML instead.
- No pilot data exist and you do not intend to set parameters manually — the workflow
  depends on per-feature distributions computed from a pilot matrix.
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: MultiPower
  role: R package providing CohenFilter() to compute Cohen's d and flag pseudo-DE
    features, and optimalRep() to consume the percentile-derived parameter object
    for sample-size estimation
  repo: github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:64bd2f6a2937838a5842731bc0dee0752fe7ab20abbd58d0e79d1b1af0ec7469
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/MultiPower
  tools:
  - MultiPower
  edam_operation: http://edamontology.org/operation_3233
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# percentile-based-parameter-estimation

## Summary

Derives MultiPower's input power parameters (variability, effect size, mean counts, dispersion, fold-change) from a preprocessed pilot multi-omic matrix by computing Cohen's d to define a pseudo-DE feature set M1, then summarizing pooled standard deviation and related parameters as Pk/P100-k percentiles over those features.

## When to use

Use when designing a multi-omic experiment and you need MultiPower input parameters estimated from pilot data: a count matrix per omic plus a Cohen's d threshold d0 are available, and sample size will be computed via optimalRep(parameters, ...).

## When NOT to use

- Input data are not yet preprocessed or still contain batch effects — MultiPower requires that data 'should have been properly preprocessed and eventual batch effects, removed' and offers no handling within the tool.
- The goal is sample-size estimation for machine learning classification — MultiPower's significance threshold and effect size parameters are not applicable there; use MultiML instead.
- No pilot data exist and you do not intend to set parameters manually — the workflow depends on per-feature distributions computed from a pilot matrix.

## Inputs

- Preprocessed, batch-effect-free pilot count matrix for each omic data type (loaded in R)
- Cohen's d effect-size threshold d0 for defining the pseudo-DE set M1
- Percentile(s) k and 100-k at which to summarize each parameter distribution

## Outputs

- Parameter object (percentile summaries of SD, variance, mean counts, dispersion, fold-change per omic) suitable for optimalRep(parameters, ...)
- cohens_d_parameters.csv with all parameter percentile fields populated and finite

## How to apply

Load the preprocessed, batch-effect-free pilot count matrix for each omic data type into R and call CohenFilter(data, d, parameters) to compute Cohen's d for all features, flagging as the pseudo-DE set M1 those features with d greater than the supplied d0 threshold. Restrict all downstream parameter computations to M1 features, computing pooled standard deviation and variance, mean counts, dispersion, and fold-change per omic. Summarize each parameter as the Pk and P100-k percentiles of its per-feature distribution, so users set a percentile of the per-feature PSD (pooled standard deviation) distribution as the dispersion parameter for power estimation. Write the resulting table to cohens_d_parameters.csv and pass it to optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect). Rationale: estimates are optimally calculated from pilot data, though they can also be manually provided; restricting to pseudo-DE features prevents undifferentiated features from inflating the dispersion estimates used in power calculations.

## Related tools

- **MultiPower** (R package providing CohenFilter() to compute Cohen's d and flag pseudo-DE features, and optimalRep() to consume the percentile-derived parameter object for sample-size estimation) — github.com/ConesaLab/MultiPower

## Examples

```
devtools::install_github("ConesaLab/MultiPower"); parameters <- CohenFilter(data, d, parameters); opt <- optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect)
```

## Evaluation signals

- cohens_d_parameters.csv contains every expected parameter field (SD, variance, mean counts, dispersion, fold-change percentiles) and all values are finite (no NA/NaN/Inf).
- All percentile summaries are computed only over the pseudo-DE set M1 (features with d > d0), not over all features.
- Per-feature pooled standard deviation matches the pooled SD recomputed directly from the pilot count matrix for a spot-checked feature.
- The parameter object is accepted by optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect) without schema errors.
- Percentile values are consistent with the chosen Pk/P100-k levels (e.g., swapping k and 100-k swaps the reported dispersion between low and high ends of the M1 distribution).

## Limitations

- Input data must be properly preprocessed and batch effects removed beforehand; the tool performs no preprocessing itself.
- Parameter estimates are optimally calculated from pilot data; without pilot data, users must provide parameters manually.
- MultiPower's parameterization (significance threshold, effect size) is not directly applicable to ML methods for sample classification; MultiML is the complementary method and itself requires pilot data.

## Evidence

- [methods] CohenFilter(data, d, parameters): "- `CohenFilter(data, d, parameters)`"
- [methods] optimalRep signature: "- `optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect)`"
- [discussion] Pilot data rationale: "Estimates for MultiPower parameters are optimally calculated from pilot data, although they can also be manually provided."
- [discussion] Preprocessing prerequisite: "Data should have been properly preprocessed and eventual batch effects, removed."
- [readme] Package installation: "devtools::install_github(“ConesaLab/MultiPower”)"
