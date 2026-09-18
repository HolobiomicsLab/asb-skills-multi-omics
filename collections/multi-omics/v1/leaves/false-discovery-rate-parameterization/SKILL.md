---
name: false-discovery-rate-parameterization
description: Use when performing power or sample-size calculations for high-throughput
  omic data (e.g. differential feature detection across m features) where a target
  FDR such as alpha = 0.05 must be translated into an adjusted per-test significance
  level alpha.
when_to_use_negative:
- When analyzing ML-based sample classification in multi-omics, where 'the basic parameters
  of the power calculation—significance threshold and effect' are not applicable (use
  MultiML instead).
- When pilot data are available and MultiPower can estimate all power parameters directly,
  making a manual alpha* computation unnecessary.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: MultiPower
  role: Reference implementation whose optimalRep fdr-based calculation adapts the
    per-test significance level for multiple testing during power/sample-size estimation
    in multi-omic experiments
  repo: github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:022d70b743303421ecbdb6decb13800cb23c7826b048c6b9b4afafd89e12ff17
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/MultiPower
  tools:
  - MultiPower
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# false-discovery-rate-parameterization

## Summary

Adapts the per-feature significance level (alpha*) for power calculations in high-throughput omic experiments, so that a target false discovery rate is respected when many features (m) are tested simultaneously. This mirrors the fdr-based adjustment used inside MultiPower's optimalRep calculation.

## When to use

Use when performing power or sample-size calculations for high-throughput omic data (e.g. differential feature detection across m features) where a target FDR such as alpha = 0.05 must be translated into an adjusted per-test significance level alpha*.

## When NOT to use

- When analyzing ML-based sample classification in multi-omics, where 'the basic parameters of the power calculation—significance threshold and effect' are not applicable (use MultiML instead).
- When pilot data are available and MultiPower can estimate all power parameters directly, making a manual alpha* computation unnecessary.

## Inputs

- Scalar m: number of features assessed simultaneously
- Scalar m1: expected number of differential (DE) features, with m1 < m
- Scalar r1: expected number of true detections
- Scalar alpha: target FDR level (e.g. 0.05, 0.01, 0.10)
- Parameter grid over m, m1, r1, alpha (data.frame for vectorized evaluation)

## Outputs

- Scalar or vector alpha*: FDR-adjusted per-feature significance level
- adjusted_alpha_table.csv with columns m, m1, r1, alpha, alpha_star

## How to apply

Write a function computing alpha* = r1*alpha / ((m - m1)(1 - alpha)) from four scalars: m (total features), m1 (expected differential features, m1 < m), r1 (expected true detections), and the desired FDR alpha (e.g. 0.05, with neighboring values 0.01 and 0.10). Build a parameter grid over these values, vectorize the function with base R, data.table, or dplyr, and evaluate alpha* for every combination. Sanity-check that alpha* is positive and smaller than the nominal alpha whenever m1 < m, since power diminishes under multiple-testing correction. Write the full grid as columns m, m1, r1, alpha, alpha_star to adjusted_alpha_table.csv. This adaptation is the significance-level step embedded in MultiPower's optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect) fdr-based calculation.

## Related tools

- **MultiPower** (Reference implementation whose optimalRep fdr-based calculation adapts the per-test significance level for multiple testing during power/sample-size estimation in multi-omic experiments) — github.com/ConesaLab/MultiPower

## Examples

```
alpha_star <- function(m, m1, r1, alpha) r1*alpha/((m-m1)*(1-alpha)); grid$alpha_star <- with(grid, alpha_star(m, m1, r1, alpha))
```

## Evaluation signals

- alpha* is strictly positive for all grid combinations where m1 < m and 0 < alpha < 1.
- alpha* < nominal alpha whenever m1 < m, consistent with power diminishing under multiple-testing correction.
- As m1 approaches m (fewer true non-differential features), alpha* increases, approaching the nominal alpha.
- Output table has exactly the columns m, m1, r1, alpha, alpha_star and one row per grid combination.
- Recomputing alpha* for a spot-checked grid cell by hand matches the vectorized value.

## Limitations

- MultiPower parameter estimates (m, m1, r1) are optimally calculated from pilot data; without pilot data users must set them manually, and poor choices propagate into alpha*.
- Input data should be properly preprocessed and batch effects removed before any MultiPower-based power calculation; the tool does not handle this itself.
- Power diminishes as the number of features increases due to multiple testing corrections, so alpha* can become extremely small for large m.

## Evidence

- [methods] In omic power calculations, the per-test significance level is adapted to account for the multiple testing scenario: [evidence span withheld — anchor status: not_found]
- [discussion] Power diminishes as the number of features increases due to the application of multiple testing corrections to control false positives.: "Power diminishes as the number of features increases due to the application of multiple testing corrections to control false positives."
- [readme] devtools::install_github(“ConesaLab/MultiPower”): "devtools::install_github(“ConesaLab/MultiPower”)"
- [methods] optimalRep fdr-based calculation: "optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect)"
