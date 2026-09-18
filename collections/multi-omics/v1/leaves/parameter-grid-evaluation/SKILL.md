---
name: parameter-grid-evaluation
description: Use when a power-calculation or correction formula depends on several
  scalar parameters (e.g., feature count m, DE features m1, expected true detections
  r1, target FDR alpha) and you need its value over all combinations rather than a
  single point estimate.
when_to_use_negative:
- The analysis goal is sample-size estimation from pilot data via the full MultiPower/optimalRep
  pipeline rather than evaluating a single scalar formula over a grid
- The parameter estimates require pilot-data-derived distributions (e.g., MultiPower
  effect-size and variability estimates), which cannot be supplied as fixed grid scalars
- The task is machine-learning sample-size estimation (MultiML), where the significance-threshold
  concept is not applicable
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MultiPower
  role: Source of the optimalRep fdr-based calculation whose alpha* significance-level
    adaptation is reproduced and grid-evaluated
  repo: https://github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:38851f7c9bcbd896e16206f337d1a393817e3de93167ce5af88d6f3f6b344baa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/MultiPower
  tools:
  - MultiPower
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# parameter-grid-evaluation

## Summary

Systematically evaluates a scalar formula or tool parameterization across a vectorized grid of input combinations — e.g., the FDR-adjusted significance level alpha* = r1*alpha / ((m - m1)(1 - alpha)) used inside MultiPower's optimalRep power calculation — and writes the full result table for inspection and sanity checking.

## When to use

Use when a power-calculation or correction formula depends on several scalar parameters (e.g., feature count m, DE features m1, expected true detections r1, target FDR alpha) and you need its value over all combinations rather than a single point estimate.

## When NOT to use

- The analysis goal is sample-size estimation from pilot data via the full MultiPower/optimalRep pipeline rather than evaluating a single scalar formula over a grid
- The parameter estimates require pilot-data-derived distributions (e.g., MultiPower effect-size and variability estimates), which cannot be supplied as fixed grid scalars
- The task is machine-learning sample-size estimation (MultiML), where the significance-threshold concept is not applicable

## Inputs

- Scalar grid values for number of features m
- Scalar grid values for expected differential features m1 (with m1 < m)
- Scalar grid values for expected true detections r1
- Target FDR levels alpha (e.g., 0.05, 0.01, 0.10)

## Outputs

- Vectorized table of alpha_star values for every grid combination
- CSV file (adjusted_alpha_table.csv) with columns m, m1, r1, alpha, alpha_star

## How to apply

Write the target quantity as a function of its scalar arguments — for the MultiPower fdr-based significance adaptation, alpha* = r1*alpha / ((m - m1)(1 - alpha)) given m features, m1 differential features (m1 < m), r1 expected true detections, and desired FDR alpha. Build a parameter grid covering realistic feature counts, DE-feature counts smaller than m, true-detection counts, and FDR targets such as alpha = 0.05 plus neighbors like 0.01 and 0.10. Vectorize the function with R (base R, data.table, or dplyr) so every grid combination is evaluated in one pass, mirroring the significance-level adaptation inside MultiPower's optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect). Sanity-check each output: alpha* must be positive and strictly smaller than the nominal alpha whenever m1 < m, reflecting that power diminishes under multiple-testing correction as feature counts grow. Finally write the complete grid (columns m, m1, r1, alpha, alpha_star) to a CSV such as adjusted_alpha_table.csv.

## Related tools

- **MultiPower** (Source of the optimalRep fdr-based calculation whose alpha* significance-level adaptation is reproduced and grid-evaluated) — https://github.com/ConesaLab/MultiPower

## Examples

```
R: alpha_star <- function(m, m1, r1, alpha) r1 * alpha / ((m - m1) * (1 - alpha)); adjusted <- grid[, alpha_star := alpha_star(m, m1, r1, alpha)]
```

## Evaluation signals

- Every computed alpha_star is positive and strictly smaller than the corresponding nominal alpha when m1 < m
- The output CSV contains exactly one row per grid combination with the required schema (m, m1, r1, alpha, alpha_star) and no missing values
- Grid explicitly covers the target FDR level alpha = 0.05 plus neighboring values such as 0.01 and 0.10
- Vectorized evaluation reproduces the scalar formula alpha* = r1*alpha / ((m - m1)(1 - alpha)) when spot-checked on individual grid points
- alpha_star decreases as m (the feature count) increases at fixed m1, r1, and alpha, consistent with multiple-testing burden

## Limitations

- Power diminishes as the number of features increases due to multiple-testing corrections, so grid values for m must reflect the actual feature scale or conclusions will be misleading
- MultiPower parameter estimates (and hence realistic grid ranges) are optimally calculated from pilot data; without pilot data they must be set manually
- The alpha* formula presumes m1 < m and a valid target FDR; degenerate grids (m1 = m) yield division-by-zero or invalid corrected levels

## Evidence

- [methods] optimalRep formula and role: "optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect)"
- [discussion] Power decreases with feature count under multiple-testing correction: "Power diminishes as the number of features increases due to the application of multiple testing corrections to control false positives."
- [discussion] Parameter estimates depend on pilot data or manual specification: "Estimates for MultiPower parameters are optimally calculated from pilot data, although they can also be manually provided."
- [readme] MultiPower is an installable R package: "MultiPower is available as an R package, and can be installed as follows"
