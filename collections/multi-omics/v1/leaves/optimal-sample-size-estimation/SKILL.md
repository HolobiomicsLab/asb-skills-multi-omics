---
name: optimal-sample-size-estimation
description: Use when designing a multi-omic study (or evaluating an existing one)
  and you need the number of samples required to reach a target power for differential
  feature detection across two or more omic platforms, given pilot data.
when_to_use_negative:
- When the goal is sample size for machine-learning sample classification — MultiPower's
  significance-threshold/effect-size parameters are not applicable there; use MultiML
  instead (which itself requires pilot data)
- When no pilot data are available and parameters cannot be manually estimated — MultiPower
  parameter estimates are optimally calculated from pilot data
- When input data are raw or contain batch effects — data must be properly preprocessed
  with batch effects removed before applying MultiPower
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0091
tools:
- name: MultiPower
  role: 'Core R package: estimates parameters from pilot data, computes per-omic power,
    and derives the optimal sample size via MultiPower(), optimalRep(), postMultiPower(),
    and powerPlot()'
  repo: github.com/ConesaLab/MultiPower
- name: FDRsampsize
  role: R dependency of MultiPower installed via install.packages() before running
    the power calculations
  repo: https://github.com/ConesaLab/MultiPower
- name: lpSolve
  role: R dependency of MultiPower installed via install.packages() before running
    the power calculations
  repo: https://github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:4c942a03566bde7399d768ff2481ee6de72f2d15b1788e613f8004669622a245
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FDRsampsize
  - MultiPower
  - lpSolve
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# optimal-sample-size-estimation

## Summary

Estimate the optimal sample size for a multi-omic differential-analysis experiment using the MultiPower R package, which computes per-omic and average statistical power from pilot data and identifies the limiting omic and sample size for a target power (e.g., 90%) at a given FDR.

## When to use

Use when designing a multi-omic study (or evaluating an existing one) and you need the number of samples required to reach a target power for differential feature detection across two or more omic platforms, given pilot data.

## When NOT to use

- When the goal is sample size for machine-learning sample classification — MultiPower's significance-threshold/effect-size parameters are not applicable there; use MultiML instead (which itself requires pilot data)
- When no pilot data are available and parameters cannot be manually estimated — MultiPower parameter estimates are optimally calculated from pilot data
- When input data are raw or contain batch effects — data must be properly preprocessed with batch effects removed before applying MultiPower

## Inputs

- Preprocessed, batch-effect-free pilot data matrices per omic platform (features x samples)
- Sample-to-group assignment vector (biological groups)
- Omic type labels and per-omic power targets (omicPower, averagePower)
- Design parameters: FDR threshold, cost constraints, equal group size flag, max sample size (max.size)

## Outputs

- Optimal sample size per omic and globally (e.g., n=24 complete TCGA GBM design; n=22 after methylation filtering)
- Identified limiting omic platform (e.g., Methyl-seq)
- Power curves via powerPlot()
- Summary table of per-omic and average optimal sample sizes (e.g., gbm_multipower_sample_size.csv)

## How to apply

Start with properly preprocessed, batch-effect-free pilot data: one feature-by-sample matrix per omic platform plus a vector assigning samples to biological groups. Install MultiPower from GitHub (devtools::install_github("ConesaLab/MultiPower")) along with its FDRsampsize and lpSolve dependencies, then estimate MultiPower's input parameters from the pilot data (parameters can also be set manually) and call MultiPower(data, groups, type, omicPower, averagePower, null.effect, fdr, cost, equalSize, max.size, omicCol, powerPlots) on the complete multi-omic design. Use optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect) to derive the optimal replicate number per omic and globally, and postMultiPower(optResults, max.size, omicCol) to summarize results; the omic with the largest number of features (e.g., Methyl-seq in the TCGA Glioblastoma case) typically drives the global sample size because multiple-testing correction reduces power as feature count grows. Decision point: feature filtering can lower the requirement — e.g., keeping only methylation sites with absolute log2 fold change > 0.05 reduced the optimal size from 24 to 22 in the TCGA GBM example. Finally, visualize power curves with powerPlot(parameters, optimalSampleSize, omicCol) and export per-omic and average optimal sample sizes (e.g., to a CSV).

## Related tools

- **MultiPower** (Core R package: estimates parameters from pilot data, computes per-omic power, and derives the optimal sample size via MultiPower(), optimalRep(), postMultiPower(), and powerPlot()) — github.com/ConesaLab/MultiPower
- **FDRsampsize** (R dependency of MultiPower installed via install.packages() before running the power calculations) — https://github.com/ConesaLab/MultiPower
- **lpSolve** (R dependency of MultiPower installed via install.packages() before running the power calculations) — https://github.com/ConesaLab/MultiPower

## Examples

```
devtools::install_github("ConesaLab/MultiPower"); res <- MultiPower(data, groups, type, omicPower, averagePower, null.effect, fdr, cost, equalSize, max.size, omicCol, powerPlots); opt <- optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect); postMultiPower(opt, max.size, omicCol)
```

## Evaluation signals

- Accept the run when the complete-design optimal sample size matches the expected limiting platform's estimate (e.g., equals 24 with Methyl-seq driving it for the TCGA Glioblastoma four-platform design)
- After filtering methylation sites to |log2 fold change| > 0.05, the optimal sample size must drop (e.g., from 24 to 22), confirming filtering reduces the power requirement
- Per-omic and average power values must meet the specified omicPower/averagePower targets at the chosen fdr within the max.size constraint
- powerPlot() produces power curves for each omic and the average power, and the exported CSV contains per-omic and average optimal sample sizes
- All dependencies (FDRsampsize, lpSolve) are installed and devtools::install_github("ConesaLab/MultiPower") completes without error before running

## Limitations

- Power diminishes as the number of features increases due to multiple testing corrections controlling false positives, so high-feature omics dominate the sample-size requirement
- MultiPower parameter estimates depend on pilot data availability; otherwise users must set parameters manually
- Input data must be properly preprocessed and batch effects removed before applying MultiPower — the tool does not handle this
- MultiPower is not directly applicable to ML methods for sample classification because the significance threshold and effect size parameters do not apply

## Evidence

- [abstract] MultiPower estimates and assesses the optimal sample size in a multi-omics experiment: "MultiPower supports different experimental settings, data types and sample sizes, and includes graphical for experimental design decision-making"
- [methods] Power calculation workflow functions: "MultiPower(data, groups, type, omicPower, averagePower, null.effect, fdr, cost, equalSize, max.size, omicCol, powerPlots)"
- [methods] Optimal replicate derivation: "optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect)"
- [readme] Installation procedure: "devtools::install_github(“ConesaLab/MultiPower”)"
- [readme] Dependency requirements: "Some dependencies are required before running MultiPower that can be installed from R via `install.packages()` and loaded with `library()`"
- [discussion] Feature-count effect on power: "Power diminishes as the number of features increases due to the application of multiple testing corrections to control false positives."
- [discussion] Pilot data and preprocessing prerequisites: "Data should have been properly preprocessed and eventual batch effects, removed."
- [discussion] ML scope boundary: "The MultiPower approach is not directly applicable to ML methods used for sample classification, as in this case the basic parameters of the power calculation—significance threshold and effect"
