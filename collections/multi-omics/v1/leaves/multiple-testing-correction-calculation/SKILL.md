---
name: multiple-testing-correction-calculation
description: 'Use when performing power calculations over many omic features where
  a target FDR (e.g. alpha = 0.05) must be controlled: the per-test significance level
  must be shrunk from nominal alpha to alpha before computing power or optimal sample
  size.'
when_to_use_negative:
- Applying machine learning methods for sample classification, where MultiPower's
  basic power parameters — significance threshold and effect size — are not applicable
  (use MultiML instead)
- Single-hypothesis testing with no multiplicity, where nominal alpha can be used
  directly and no FDR adaptation of the per-test level is needed
- Feature-level differential expression testing itself rather than experimental design/power
  analysis — alpha* here is an input to power calculations, not a test statistic threshold
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MultiPower
  role: Source of the fdr-based optimalRep calculation whose significance-level adaptation
    this skill reimplements as an explicit alpha* formula over a parameter grid
  repo: github.com/ConesaLab/MultiPower
- name: FDRsampsize
  role: R dependency of MultiPower relevant to FDR-based sample size computation,
    installed via install.packages() before running MultiPower
  repo: https://github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:288c7593589fe815190fa492ec8e4801a8a38faeeb4c51056c1ed20ca8b7b8d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/MultiPower
  tools:
  - FDRsampsize
  - MultiPower
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: thin
yes_rate: 0.0
n_claims: 0
---

# multiple-testing-correction-calculation

## Summary

Computes the FDR-adjusted per-feature significance level alpha* = r1*alpha / ((m - m1)(1 - alpha)) used inside MultiPower's optimalRep fdr-based power calculation, so that per-test power calculations in high-throughput omic experiments respect a target false discovery rate when m features are assessed simultaneously.

## When to use

Use when performing power calculations over many omic features where a target FDR (e.g. alpha = 0.05) must be controlled: the per-test significance level must be shrunk from nominal alpha to alpha* before computing power or optimal sample size.

## When NOT to use

- Applying machine learning methods for sample classification, where MultiPower's basic power parameters — significance threshold and effect size — are not applicable (use MultiML instead)
- Single-hypothesis testing with no multiplicity, where nominal alpha can be used directly and no FDR adaptation of the per-test level is needed
- Feature-level differential expression testing itself rather than experimental design/power analysis — alpha* here is an input to power calculations, not a test statistic threshold

## Inputs

- Scalar m: number of features assessed
- Scalar m1: expected number of differential (DE) features, with m1 < m
- Scalar r1: expected number of true detections
- Scalar alpha: target FDR level (0.05, plus neighbors such as 0.01 and 0.10)

## Outputs

- Scalar or vector alpha*: FDR-adjusted per-feature significance level
- CSV table adjusted_alpha_table.csv with columns m, m1, r1, alpha, alpha_star covering the full parameter grid

## How to apply

Write a scalar function alpha* = r1*alpha / ((m - m1)(1 - alpha)) taking the number of features m, expected differential features m1 < m, expected true detections r1, and the desired FDR alpha (e.g. 0.05, with neighbors 0.01 and 0.10), mirroring the significance-level adaptation inside MultiPower's optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect). Build a parameter grid over m, m1, r1, and alpha, vectorize the function (e.g. with base R data.table or dplyr) to evaluate alpha* for every combination, and write the result with columns m, m1, r1, alpha, alpha_star to adjusted_alpha_table.csv. The rationale is that power diminishes as the number of features increases due to multiple testing corrections controlling false positives, so the nominal level must be adapted per test for the expected proportion of false discoveries to be controlled.

## Related tools

- **MultiPower** (Source of the fdr-based optimalRep calculation whose significance-level adaptation this skill reimplements as an explicit alpha* formula over a parameter grid) — github.com/ConesaLab/MultiPower
- **FDRsampsize** (R dependency of MultiPower relevant to FDR-based sample size computation, installed via install.packages() before running MultiPower) — https://github.com/ConesaLab/MultiPower

## Examples

```
alpha_star <- function(m, m1, r1, alpha) r1 * alpha / ((m - m1) * (1 - alpha)); adjusted_alpha_table.csv <- expand.grid(m=1000, m1=100, r1=50, alpha=0.05)
```

## Evaluation signals

- Sanity-check that alpha* is positive for every grid combination
- Verify alpha* < nominal alpha whenever m1 < m, consistent with power diminishing under multiple-testing correction
- Check the output table has exactly the columns m, m1, r1, alpha, alpha_star and one row per grid combination
- Confirm alpha* is a strictly increasing function of r1 and of m1 (more expected detections relax the per-test level)
- Cross-check the formula against MultiPower's fdr-based optimalRep behavior for identical (m, m1, r1, alpha) inputs

## Limitations

- Requires pilot-data-derived or manually provided parameter estimates: m, m1, and r1 must be supplied since MultiPower parameter estimates optimally come from pilot data but can also be manually provided
- Input omic data must be properly preprocessed and batch-effect free before the enclosing MultiPower analysis; the tool itself does not handle preprocessing
- Not applicable to ML sample classification problems, where significance threshold and effect size do not apply (use MultiML)

## Evidence

- [abstract] adjusted alpha* derivation: "we formulate the MultiPower method to estimate and assess the optimal sample size in a multi-omics experiment"
- [discussion] rationale for level adaptation: "Power diminishes as the number of features increases due to the application of multiple testing corrections to control false positives."
- [readme] installation context: "devtools::install_github(“ConesaLab/MultiPower”)"
