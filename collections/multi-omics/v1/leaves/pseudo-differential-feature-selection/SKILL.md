---
name: pseudo-differential-feature-selection
description: Use when a preprocessed, batch-effect-free pilot multi-omic count matrix
  is available and you need to derive MultiPower inputs (variability/PSD percentile,
  effect size, number of variables) for optimal sample-size estimation.
when_to_use_negative:
- Input data still contains batch effects or has not been properly preprocessed —
  MultiPower requires batch effects to be removed beforehand and does not handle them
  internally
- No pilot data are available and no estimates can be computed — parameters must then
  be manually provided instead
- The goal is ML classification sample-size estimation (use MultiML), since MultiPower's
  significance threshold and effect size parameters are not applicable there
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_2269
tools:
- name: MultiPower
  role: Provides CohenFilter(data, d, parameters) to compute Cohen's d and derive
    the pseudo-DE set M1 and the percentile-summarized parameters for power calculation
  repo: github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:fa24682700914c229447cffb86c061c1fde9f39226f3b904db2bf96cd805433f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiPower
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# pseudo-differential-feature-selection

## Summary

Selects a pseudo-differentially expressed (pseudo-DE) feature set M1 in MultiPower by computing Cohen's d per feature with CohenFilter(data, d, parameters) and flagging features whose d exceeds the supplied d0 threshold; parameters for sample-size estimation are then summarized as percentiles over M1.

## When to use

Use when a preprocessed, batch-effect-free pilot multi-omic count matrix is available and you need to derive MultiPower inputs (variability/PSD percentile, effect size, number of variables) for optimal sample-size estimation.

## When NOT to use

- Input data still contains batch effects or has not been properly preprocessed — MultiPower requires batch effects to be removed beforehand and does not handle them internally
- No pilot data are available and no estimates can be computed — parameters must then be manually provided instead
- The goal is ML classification sample-size estimation (use MultiML), since MultiPower's significance threshold and effect size parameters are not applicable there

## Inputs

- Preprocessed, batch-effect-free pilot count matrix for each omic data type (R data object)
- Cohen's d effect-size threshold d0
- Percentile k for summarizing per-feature parameter distributions (Pk and P100-k)

## Outputs

- Pseudo-DE feature set M1 (features with Cohen's d > d0)
- Parameter table (e.g. cohens_d_parameters.csv) with SD, variance, mean counts, dispersion, and fold-change percentiles per omic
- parameters object suitable as input to optimalRep(parameters, ...)

## How to apply

Load the preprocessed, batch-effect-free pilot count matrix for each omic data type into R and call the exported MultiPower function CohenFilter(data, d, parameters), which computes Cohen's d for all features and flags the pseudo-DE set M1 as features with d greater than the supplied d0 threshold. Over the M1 features only, compute the pooled standard deviation and variance (the variability input for power), mean counts, dispersion, and fold-change per omic. Summarize each parameter as the Pk and P100-k percentiles over the pseudo-DE features, producing the parameters object expected by optimalRep(parameters, ...). The rationale is that MultiPower parameter estimates are optimally calculated from pilot data — the pseudo-DE filtering restricts parameter estimation to features likely to be of biological interest. Write the parameter table (e.g. cohens_d_parameters.csv) and validate that every field (SD, variance, mean counts, dispersion, fold-change percentiles) is present and finite before downstream sample-size calculation.

## Related tools

- **MultiPower** (Provides CohenFilter(data, d, parameters) to compute Cohen's d and derive the pseudo-DE set M1 and the percentile-summarized parameters for power calculation) — github.com/ConesaLab/MultiPower

## Examples

```
install.packages("devtools"); devtools::install_github("ConesaLab/MultiPower"); parameters <- CohenFilter(data, d = d0)
```

## Evaluation signals

- M1 contains only features whose Cohen's d exceeds the supplied d0 threshold
- cohens_d_parameters.csv contains every expected field: SD, variance, mean counts, dispersion, and fold-change percentiles
- All parameter fields are finite (no NA/Inf values) after percentile summarization
- Each parameter is summarized as both the Pk and P100-k percentiles over the pseudo-DE features
- The resulting parameters object is accepted by optimalRep(parameters, ...) without errors

## Limitations

- Input data must be properly preprocessed and batch effects removed before applying MultiPower; the tool provides no handling for this
- Parameter estimates are optimally calculated from pilot data; without pilot data users must set parameters manually
- Power diminishes as the number of features increases due to multiple testing corrections controlling false positives

## Evidence

- [methods] CohenFilter computes Cohen's d and flags pseudo-DE set: "CohenFilter(data, d, parameters)"
- [methods] optimalRep consumes the parameters object: "optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect)"
- [discussion] Pilot data requisite and manual parameter option: "Estimates for MultiPower parameters are optimally calculated from pilot data, although they can also be manually provided."
- [discussion] Preprocessing/batch-effect prerequisite: "Data should have been properly preprocessed and eventual batch effects, removed."
- [readme] MultiPower purpose from README: "The MultiPower R method performs statistical power studies for multi-omics experiments, and is designed to assist users in experimental design"
