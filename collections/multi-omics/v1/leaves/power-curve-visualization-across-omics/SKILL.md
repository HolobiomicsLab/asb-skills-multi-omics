---
name: power-curve-visualization-across-omics
description: Use when a multi-omics pilot dataset has been processed with MultiPower/optimalRep
  and per-omic power or detectable Cohen's d trade-off curves are needed to inspect
  or communicate optimal sample sizes.
when_to_use_negative:
- Pilot data have not been preprocessed or batch effects have not been removed — MultiPower
  parameter estimates and hence curves would be invalid
- The goal is sample-size estimation for machine learning classification (use MultiML
  instead; MultiPower's significance-threshold/effect-size parameters are not applicable
  to ML classification)
- No pilot data are available and parameters cannot be estimated or manually supplied
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: MultiPower
  role: R package providing powerPlot(parameters, optimalSampleSize, omicCol) and
    postMultiPower(optResults, max.size, omicCol) to generate per-omic power and trade-off
    curves
  repo: github.com/ConesaLab/MultiPower
merged_aliases:
- power-curve-plot-visualization-across-omics
merged_alias_records:
- alias: power-curve-plot-visualization-across-omics
  slug: power-curve-plot-visualization-across-omics
  jaccard_score: 0.8333
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:eaf452954bf6a717caac60526008b8759e506f896c913b9438112c24e83f6dec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/MultiPower
  tools:
  - MultiPower
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# power-curve-visualization-across-omics

## Summary

Render sample-size vs. power (and sample-size vs. detectable effect-size) curves for multi-omics experiments using MultiPower's powerPlot, enabling visual inspection of power relationships across RNA-seq, miRNA-seq, ChIP-seq, DNase-seq, metabolomics and proteomics platforms.

## When to use

Use when a multi-omics pilot dataset has been processed with MultiPower/optimalRep and per-omic power or detectable Cohen's d trade-off curves are needed to inspect or communicate optimal sample sizes.

## When NOT to use

- Pilot data have not been preprocessed or batch effects have not been removed — MultiPower parameter estimates and hence curves would be invalid
- The goal is sample-size estimation for machine learning classification (use MultiML instead; MultiPower's significance-threshold/effect-size parameters are not applicable to ML classification)
- No pilot data are available and parameters cannot be estimated or manually supplied

## Inputs

- Parameters object from MultiPower/MultiGroupPower power estimation on preprocessed, batch-effect-free multi-omic matrices (e.g. STATegra RNA-seq, miRNA-seq, ChIP-seq, DNase-seq, metabolomics, proteomics)
- Optimal sample-size object from optimalRep or postMultiPower (per-omic sizes, e.g. 17/17/9/6 under equal cost)
- omicCol color mapping vector assigning a color per omic platform

## Outputs

- Power curve plot (sample size vs. per-omic power) rendered to file, e.g. multipower power curves per platform
- Sample-size vs. detectable effect-size trade-off curve, e.g. postmultipower_tradeoff_curve.png
- Summary table of per-omic optimal sample sizes and power values (e.g. multipower_results.csv)

## How to apply

After installing MultiPower from GitHub (ConesaLab/MultiPower) and its CRAN dependencies FDRsampsize and lpSolve, run MultiPower (or MultiGroupPower) on the preprocessed, batch-effect-free omic matrices with parameters such as fdr = 0.05, omicPower = 0.6, averagePower = 0.8, and equalSize set per design. Feed the optimization result to postMultiPower(optResults, max.size, omicCol) to obtain per-omic power at the optimal sample size or the sample-size vs. detectable effect-size trade-off. Then call powerPlot(parameters, optimalSampleSize, omicCol) to draw per-platform power curves, supplying omicCol to assign each omic a distinct color so RNA-seq, miRNA-seq, ChIP-seq, DNase-seq, metabolomics and proteomics curves are visually separable. Write the resulting figure (e.g. postmultipower_tradeoff_curve.png) alongside a table of per-omic optimal sample sizes, and validate that curves reproduce expected values such as Cohen's d = 1.98 at n = 4 or per-omic powers matching Table 1 (e.g. RNA-seq 0.999, DNase-seq 0.627 at n = 16).

## Related tools

- **MultiPower** (R package providing powerPlot(parameters, optimalSampleSize, omicCol) and postMultiPower(optResults, max.size, omicCol) to generate per-omic power and trade-off curves) — github.com/ConesaLab/MultiPower

## Examples

```
library(MultiPower); opt <- MultiPower(data, groups, type, omicPower=0.6, averagePower=0.8, fdr=0.05, equalSize=TRUE); p <- postMultiPower(opt, omicCol=omicCol); powerPlot(p$parameters, p$optimalSampleSize, omicCol)
```

## Evaluation signals

- Trade-off curve reports a detectable Cohen's d of 1.98 at n = 4 samples per condition for the STATegra unequal-size scenario
- Per-omic optimal sizes shown/annotated match the expected values (e.g. 17 miRNA-seq, 17 DNase-seq, 9 RNA-seq, 6 metabolomics at equal cost)
- At the equal-size optimum n = 16, per-omic powers match Table 1: RNA-seq 0.999, miRNA-seq 0.680, ChIP-seq 0.898, DNase-seq 0.627, metabolomics 1.000, proteomics 0.685
- Each omic platform is rendered in its assigned color from omicCol and curves are visually separable
- Plot file (e.g. postmultipower_tradeoff_curve.png) and results table (e.g. multipower_results.csv) are produced together

## Limitations

- Power diminishes as the number of features increases due to multiple testing corrections, so curves are sensitive to the feature set retained after filtering (e.g. CohenFilter with d = 0.8)
- Curves reflect parameters estimated from pilot data; without pilot data, parameters must be manually supplied
- Input data must already be preprocessed and free of batch effects; MultiPower/powerPlot do not handle this

## Evidence

- [methods] powerPlot function signature: "powerPlot(parameters, optimalSampleSize, omicCol)"
- [methods] postMultiPower computes sample-size trade-off: "postMultiPower(optResults, max.size, omicCol)"
- [readme] package install: "devtools::install_github(“ConesaLab/MultiPower”)"
- [readme] dependencies required before running: "Some dependencies are required before running MultiPower that can be installed from R via `install.packages()` and loaded with `library()`"
- [discussion] power decreases with feature number: "Power diminishes as the number of features increases due to the application of multiple testing corrections to control false positives."
- [discussion] data preprocessing prerequisite: "Data should have been properly preprocessed and eventual batch effects, removed."
