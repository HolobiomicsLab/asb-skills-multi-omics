---
name: r-statistical-programming
description: Use when designing high-throughput multi-omic experiments where many
  features are tested simultaneously and a target false discovery rate (e.g. alpha
  = 0.05, 0.01, 0.10) must be translated into a per-test significance level for power/sample-size
  calculations.
when_to_use_negative:
- The analysis targets ML sample classification (MultiML), where the article states
  the basic power parameters—significance threshold and effect size—are not applicable
- Input is a single hypothesis test or a pre-corrected set of p-values rather than
  a multiple-testing power-design scenario
- Pilot data are available for direct parameter estimation; MultiPower parameter estimates
  are optimally calculated from pilot data rather than set manually
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MultiPower
  role: R package whose fdr-based optimalRep calculation contains the alpha* adaptation
    implemented here; usable to cross-check the vectorized grid results
  repo: https://github.com/ConesaLab/MultiPower
- name: FDRsampsize
  role: R dependency installed via install.packages() before running MultiPower power
    calculations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:95244bd7449c7f01af3ea294732362b7d1d5850f70ad7bd0b7ef976f71aa4045
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/MultiPower
  tools:
  - FDRsampsize
  - MultiPower
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# r-statistical-programming

## Summary

Use R (base R, data.table/dplyr) to implement and vectorize the per-feature FDR-adjusted significance level alpha* = r1*alpha/((m-m1)(1-alpha)) used inside MultiPower's fdr-based optimalRep power calculations for multi-omic experiments.

## When to use

Use when designing high-throughput multi-omic experiments where many features are tested simultaneously and a target false discovery rate (e.g. alpha = 0.05, 0.01, 0.10) must be translated into a per-test significance level for power/sample-size calculations.

## When NOT to use

- The analysis targets ML sample classification (MultiML), where the article states the basic power parameters—significance threshold and effect size—are not applicable
- Input is a single hypothesis test or a pre-corrected set of p-values rather than a multiple-testing power-design scenario
- Pilot data are available for direct parameter estimation; MultiPower parameter estimates are optimally calculated from pilot data rather than set manually

## Inputs

- Scalar or grid of feature counts m (total features assessed)
- Scalar or grid of expected differential features m1 (with m1 < m)
- Scalar or grid of expected true detections r1
- Target false discovery rate alpha (e.g. 0.05, 0.01, 0.10)

## Outputs

- adjusted_alpha_table.csv with columns m, m1, r1, alpha, alpha_star
- Vectorized R function returning the FDR-adjusted per-feature significance level alpha*

## How to apply

Write an R function taking four scalars (m = number of features, m1 = expected differential features with m1 < m, r1 = expected true detections, alpha = target FDR) and returning alpha* = r1*alpha/((m-m1)*(1-alpha)), mirroring the significance-level adaptation used in MultiPower's optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect). Build a parameter grid over m, m1 < m, r1, and alpha values of 0.05 plus neighbors such as 0.01 and 0.10, then vectorize the computation (base R, data.table, or dplyr) across all grid combinations. Sanity-check that alpha* is positive and smaller than the nominal alpha whenever m1 < m, reflecting that power diminishes as the number of features increases under multiple-testing correction. Write the full grid with columns m, m1, r1, alpha, alpha_star to adjusted_alpha_table.csv. The MultiPower package and its dependencies (FDRsampsize, lpSolve) can be installed from R via devtools::install_github("ConesaLab/MultiPower") to cross-check against the reference implementation.

## Related tools

- **MultiPower** (R package whose fdr-based optimalRep calculation contains the alpha* adaptation implemented here; usable to cross-check the vectorized grid results) — https://github.com/ConesaLab/MultiPower
- **FDRsampsize** (R dependency installed via install.packages() before running MultiPower power calculations)

## Examples

```
alpha_star <- function(m, m1, r1, alpha) r1*alpha/((m-m1)*(1-alpha)); grid$alpha_star <- with(grid, alpha_star(m, m1, r1, alpha)); write.csv(grid, "adjusted_alpha_table.csv", row.names=FALSE)
```

## Evaluation signals

- alpha* is strictly positive for every grid combination
- alpha* < alpha (nominal FDR) whenever m1 < m, consistent with power diminishing under multiple-testing correction
- adjusted_alpha_table.csv contains exactly the columns m, m1, r1, alpha, alpha_star and one row per grid combination
- Hand-checked case: for m=10000, m1=100, r1=50, alpha=0.05, alpha* = 50*0.05/((10000-100)*(1-0.05)) ≈ 0.000265

## Limitations

- Estimates for MultiPower parameters (including m1 and r1) are optimally calculated from pilot data, although they can also be manually provided; without pilot data the grid values are assumptions
- The alpha* adaptation is specific to differential-feature power calculations and is not directly applicable to ML sample classification, where significance threshold and effect size are not applicable
- Input data must be properly preprocessed and batch-effect free before applying MultiPower; the tool provides no handling for this

## Evidence

- [discussion] Power diminishes as the number of features increases due to multiple testing corrections controlling false positives: "Power diminishes as the number of features increases due to the application of multiple testing corrections to control false positives."
- [readme] MultiPower is an R package implementing statistical power studies for multi-omics experiments: "The MultiPower R method performs statistical power studies for multi-omics experiments, and is designed to assist users in experimental design"
- [methods] MultiPower installation from GitHub using devtools: "devtools::install_github(“ConesaLab/MultiPower”)"
- [methods] optimalRep function signature adapted from the article's filter steps: "optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect)"
- [abstract] MultiPower estimates optimal sample size in a multi-omics experiment: "we formulate the MultiPower method to estimate and assess the optimal sample size in a multi-omics experiment"
