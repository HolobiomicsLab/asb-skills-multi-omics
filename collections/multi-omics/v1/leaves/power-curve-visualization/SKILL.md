---
name: power-curve-visualization
description: Use when a multi-omic power analysis (MultiPower) has been run with optimalRep()/postMultiPower()
  and power-vs-sample-size curves are needed per omic and for average power to justify
  the chosen sample size.
when_to_use_negative:
- No pilot data are available and MultiPower parameters must be set manually without
  estimates, making curves uninformative
- Input data have not been preprocessed or batch effects have not been removed, since
  MultiPower requires batch-effect-free data before power estimation and plotting
- The goal is sample-size estimation for machine learning classification (use MultiML
  instead; MultiPower power curves target differential feature analysis)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3520
tools:
- name: MultiPower
  role: R package providing powerPlot(parameters, optimalSampleSize, omicCol) for
    rendering per-omic and average power curves after optimalRep()/postMultiPower()
  repo: https://github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:22c0073779535892235e333d9a8f6b13c166f97d1d3988cecb07a966e419d63c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/MultiPower
  tools:
  - MultiPower
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# power-curve-visualization

## Summary

Generate per-omic statistical power curves for a multi-omic experiment using MultiPower's powerPlot() to visualize how power varies with sample size, supporting experimental design decisions such as confirming an optimal sample size (e.g. n=24 for TCGA Glioblastoma, reduced to n=22 after methylation filtering).

## When to use

Use when a multi-omic power analysis (MultiPower) has been run with optimalRep()/postMultiPower() and power-vs-sample-size curves are needed per omic and for average power to justify the chosen sample size.

## When NOT to use

- No pilot data are available and MultiPower parameters must be set manually without estimates, making curves uninformative
- Input data have not been preprocessed or batch effects have not been removed, since MultiPower requires batch-effect-free data before power estimation and plotting
- The goal is sample-size estimation for machine learning classification (use MultiML instead; MultiPower power curves target differential feature analysis)

## Inputs

- MultiPower parameter estimates object (output of MultiPower()/MultiGroupPower() estimated from preprocessed pilot multi-omic data)
- Optimal sample size object returned by optimalRep()/postMultiPower()
- omicCol: per-omic color vector for curve rendering

## Outputs

- Per-omic and average power curves (plots) showing power as a function of sample size
- Graphical support for design decisions, annotating the optimal sample size per omic and globally

## How to apply

After running MultiPower(data, groups, type, omicPower, averagePower, null.effect, fdr, cost, equalSize, max.size, omicCol, powerPlots) on preprocessed, batch-effect-free pilot data, derive the optimal sample size per omic with optimalRep() and postMultiPower(), then call powerPlot(parameters, optimalSampleSize, omicCol) to render power curves showing how per-omic and average power change with sample size. Color the curves per omic using the omicCol argument so each platform (e.g. TCGA Glioblastoma's four omic platforms including Methyl-seq) is distinguishable, and mark the optimal sample size (e.g. 24, or 22 after filtering methylation sites with absolute log2 fold change >0.05). The rationale is that power diminishes as the number of features increases due to multiple testing corrections, so curves identify which omic is the limiting platform and how feature filtering shifts the optimum. Compare curves before and after filtering (e.g. complete vs. methylation-filtered designs) as a visual validation that the optimal sample size moved from 24 to 22.

## Related tools

- **MultiPower** (R package providing powerPlot(parameters, optimalSampleSize, omicCol) for rendering per-omic and average power curves after optimalRep()/postMultiPower()) — https://github.com/ConesaLab/MultiPower

## Examples

```
devtools::install_github("ConesaLab/MultiPower"); powerPlot(parameters, optimalSampleSize, omicCol)
```

## Evaluation signals

- powerPlot() executes without error on the optimalSampleSize object returned by optimalRep()/postMultiPower()
- Curves show power increasing monotonically with sample size until the target average power is reached at the reported optimum (e.g. n=24)
- The limiting omic (Methyl-seq, due to its high number of features) has its curve reaching target power latest, matching the globally limiting platform
- After filtering methylation sites with absolute log2 fold change >0.05, the regenerated curves annotate a reduced optimum of n=22 versus n=24
- Curve colors match the omicCol assignment so each of the four omic platforms is visually distinct

## Limitations

- Power curves are only as reliable as the pilot-data-derived parameter estimates; MultiPower parameters should be estimated from pilot data and can otherwise be manually provided
- Input data must be properly preprocessed with eventual batch effects removed before parameter estimation and plotting
- Power diminishes as the number of features increases due to multiple testing corrections, so feature-rich omics (like Methyl-seq) can dominate and limit the design

## Evidence

- [methods] powerPlot signature: "powerPlot(parameters, optimalSampleSize, omicCol)"
- [abstract] MultiPower provides graphical support for design decisions: "MultiPower supports different experimental settings, data types and sample sizes, and includes graphical for experimental design decision-making"
- [readme] install via devtools from GitHub: "devtools::install_github(“ConesaLab/MultiPower”)"
- [discussion] power decreases with feature count: "Power diminishes as the number of features increases due to the application of multiple testing corrections to control false positives."
