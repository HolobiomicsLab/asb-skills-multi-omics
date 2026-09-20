---
name: power-analysis-methods
description: Use when designing or evaluating a differential-analysis experiment over
  many omic features (RNA-seq, proteomics, etc.) and you must compute the per-test
  alpha adjusted for a target FDR, or estimate optimal replicate numbers.
when_to_use_negative:
- The goal is sample size for machine-learning sample classification — MultiPower's
  significance threshold and effect size do not apply; use MultiML instead
- Input data are not preprocessed and batch-effect free — MultiPower does not handle
  preprocessing or batch removal internally
- The task is QC reporting rather than power/sample-size design (use harmonized Figures
  of Merit with FastQC, Qualimap, SAMstat, MultiQC)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_3577
tools:
- name: MultiPower
  role: R package whose optimalRep function performs the fdr-based power/sample-size
    calculation in which the per-test alpha* adaptation is embedded
  repo: github.com/ConesaLab/MultiPower
- name: FDRsampsize
  role: Required R dependency of MultiPower for FDR-based sample-size computations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:91c8af7906361a5b498f39220890f04e3601d0f8346d4a384e1c61173dfc2894
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FDRsampsize
  - MultiPower
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# power-analysis-methods

## Summary

Statistical power and sample-size analysis for high-throughput multi-omic experiments, exemplified by MultiPower, in which the per-feature significance level is adapted to a target FDR (alpha* = r1*alpha / ((m - m1)(1 - alpha))) because power diminishes as the number of tested features increases under multiple-testing correction.

## When to use

Use when designing or evaluating a differential-analysis experiment over many omic features (RNA-seq, proteomics, etc.) and you must compute the per-test alpha adjusted for a target FDR, or estimate optimal replicate numbers.

## When NOT to use

- The goal is sample size for machine-learning sample classification — MultiPower's significance threshold and effect size do not apply; use MultiML instead
- Input data are not preprocessed and batch-effect free — MultiPower does not handle preprocessing or batch removal internally
- The task is QC reporting rather than power/sample-size design (use harmonized Figures of Merit with FastQC, Qualimap, SAMstat, MultiQC)

## Inputs

- Scalar m: number of features tested
- Scalar m1 (< m): expected number of differential features
- Scalar r1: expected number of true detections
- Scalar alpha: target FDR level (e.g. 0.05, 0.01, 0.10)
- Parameter grid over (m, m1, r1, alpha) combinations
- Optional pilot multi-omic data for estimating MultiPower parameters

## Outputs

- Per-feature FDR-adjusted significance level alpha_star for each grid combination
- adjusted_alpha_table.csv with columns m, m1, r1, alpha, alpha_star
- optimalRep result: estimated optimal sample size for a target omicPower/averagePower at given fdr

## How to apply

Write a scalar function computing the FDR-adjusted significance level alpha* = r1*alpha / ((m - m1)(1 - alpha)) from the number of features m, expected differential features m1 < m, expected true detections r1, and the target FDR alpha (e.g. 0.05, plus neighbors such as 0.01 and 0.10); this mirrors the significance-level adaptation inside MultiPower's optimalRep fdr-based calculation. Build a parameter grid over these four scalars, vectorize the function with base R (or data.table/dplyr), and evaluate alpha* for every combination. Sanity-check that alpha* is positive and smaller than the nominal alpha whenever m1 < m, consistent with power diminishing under multiple-testing corrections that control false positives. In the full MultiPower workflow, parameter estimates are optimally obtained from pilot data via optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect), otherwise they must be set manually. Write the full grid (columns m, m1, r1, alpha, alpha_star) to adjusted_alpha_table.csv.

## Related tools

- **MultiPower** (R package whose optimalRep function performs the fdr-based power/sample-size calculation in which the per-test alpha* adaptation is embedded) — github.com/ConesaLab/MultiPower
- **FDRsampsize** (Required R dependency of MultiPower for FDR-based sample-size computations)

## Examples

```
alpha_star <- function(m, m1, r1, alpha) r1*alpha/((m-m1)*(1-alpha)); alpha_star(m=20000, m1=500, r1=400, alpha=0.05)
```

## Evaluation signals

- alpha_star is strictly positive for all grid rows
- alpha_star < nominal alpha for every row where m1 < m, reflecting power loss under multiple-testing correction
- Output CSV matches the expected schema exactly: columns m, m1, r1, alpha, alpha_star with one row per grid combination
- Grid covers the target FDR alpha = 0.05 and its neighbors (e.g. 0.01, 0.10) across varied m and m1 < m
- When run through optimalRep, the resulting optimal sample size achieves the specified fdr/omicPower targets

## Limitations

- MultiPower parameter estimates depend on pilot data availability; without pilot data users must set parameters manually
- MultiPower is not directly applicable to ML methods for sample classification because significance threshold and effect size are not applicable
- Data must be properly preprocessed and batch effects removed before applying MultiPower; no handling exists within the tool
- Power diminishes as the number of features increases due to multiple testing corrections, so per-feature alpha must be adapted rather than held at nominal levels

## Evidence

- [discussion] Power diminishes as the number of features increases due to the application of multiple testing corrections to control false positives.: "Power diminishes as the number of features increases due to the application of multiple testing corrections to control false positives."
- [readme] The MultiPower R method performs statistical power studies for multi-omics experiments: "The MultiPower R method performs statistical power studies for multi-omics experiments"
- [discussion] Estimates for MultiPower parameters are optimally calculated from pilot data, although they can also be manually provided.: "Estimates for MultiPower parameters are optimally calculated from pilot data, although they can also be manually provided."
- [methods] Some dependencies are required before running MultiPower that can be installed from R via install.packages(): "Some dependencies are required before running MultiPower that can be installed from R via install.packages()"
