---
name: effect-size-tradeoff-curve-interpretation
description: Use when a MultiPower optimization run on pilot multi-omics data (e.g.
  STATegra miRNA-seq, RNA-seq, DNase-seq, metabolomics) needs a post-hoc analysis
  of how many replicates per group achieve a target detectable Cohen's d, especially
  when equalSize=FALSE allows unequal per-omic sizes at equal cost.
when_to_use_negative:
- Input pilot data still contain batch effects or are otherwise unpreprocessed — MultiPower
  requires data to be 'properly preprocessed and eventual batch effects, removed'
  before the trade-off analysis is meaningful
- No pilot data are available and parameters cannot be estimated — MultiPower parameter
  estimates depend on pilot data availability, otherwise users must set parameters
  manually
- The goal is sample size for machine learning classification rather than differential-feature
  power — MultiPower is not directly applicable to ML methods because significance
  threshold and effect size are not applicable
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: MultiPower
  role: R package whose MultiPower() call with equalSize=FALSE and cost constraint
    produces the optimization object, and whose postMultiPower() and powerPlot() functions
    compute and render the sample-size vs Cohen's d trade-off curve
  repo: github.com/ConesaLab/MultiPower
- name: devtools
  role: Used to install the MultiPower R package from GitHub before running the analysis
  repo: github.com/ConesaLab/MultiPower
- name: FDRsampsize
  role: Required R dependency of MultiPower for power calculations
- name: lpSolve
  role: Required R dependency of MultiPower for the cost-constrained optimization
    of per-omic sample sizes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:ce48942c1811adccd686745a8c8194548783dd9f85f9902c7a41fd73057ed814
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/MultiPower
  tools:
  - FDRsampsize
  - MultiPower
  - devtools
  - lpSolve
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.25
n_claims: 8
---

# effect-size-tradeoff-curve-interpretation

## Summary

Interprets the MultiPower post-hoc trade-off between per-omic optimal sample sizes and the minimum detectable effect size (Cohen's d) in a multi-omics experiment, using postMultiPower and powerPlot to render the sample-size vs detectable-effect curve for experimental design decisions.

## When to use

Use when a MultiPower optimization run on pilot multi-omics data (e.g. STATegra miRNA-seq, RNA-seq, DNase-seq, metabolomics) needs a post-hoc analysis of how many replicates per group achieve a target detectable Cohen's d, especially when equalSize=FALSE allows unequal per-omic sizes at equal cost.

## When NOT to use

- Input pilot data still contain batch effects or are otherwise unpreprocessed — MultiPower requires data to be 'properly preprocessed and eventual batch effects, removed' before the trade-off analysis is meaningful
- No pilot data are available and parameters cannot be estimated — MultiPower parameter estimates depend on pilot data availability, otherwise users must set parameters manually
- The goal is sample size for machine learning classification rather than differential-feature power — MultiPower is not directly applicable to ML methods because significance threshold and effect size are not applicable

## Inputs

- MultiPower optimization result object (output of MultiPower()/optimalRep() with equalSize=FALSE and a cost constraint)
- Preprocessed, batch-effect-free pilot multi-omics matrices (e.g. miRNA-seq, RNA-seq, DNase-seq, metabolomics count/intensity matrices)
- Group labels and omic type annotations (e.g. RNA-seq vs metabolomics) as required by MultiPower
- Cost constraint and omic color palette (omicCol) for plotting

## Outputs

- Sample-size vs detectable effect-size trade-off curve (e.g. postmultipower_tradeoff_curve.png) rendered via powerPlot
- Table of per-omic optimal sample sizes at equal cost (e.g. 17/17/9/6 for STATegra omics)
- Minimum detectable Cohen's d values at each candidate sample size (e.g. d = 1.98 at n = 4 per condition)

## How to apply

After running MultiPower(data, groups, type, omicPower, averagePower, null.effect, fdr, cost, equalSize, max.size, omicCol, powerPlots) with equalSize=FALSE and a cost constraint so per-omic sizes are optimized at equal total cost (e.g. STATegra: 17 miRNA-seq/DNase-seq, 9 RNA-seq, 6 metabolomics per group), feed the resulting optimization object to postMultiPower(optResults, max.size, omicCol) to compute the sample-size vs detectable effect-size trade-off. Render the curve with powerPlot(parameters, optimalSampleSize, omicCol), writing the plot to e.g. postmultipower_tradeoff_curve.png alongside a table of per-omic optimal sample sizes. Read the curve to determine the minimum detectable Cohen's d at each sample size — for the STATegra reproduction the reference point is a detectable d of 1.98 at n=4 samples per condition. Accept the interpretation if the reported per-omic sizes match the equal-cost optimum and the trade-off curve reproduces the reference effect-size/sample-size pairs.

## Related tools

- **MultiPower** (R package whose MultiPower() call with equalSize=FALSE and cost constraint produces the optimization object, and whose postMultiPower() and powerPlot() functions compute and render the sample-size vs Cohen's d trade-off curve) — github.com/ConesaLab/MultiPower
- **devtools** (Used to install the MultiPower R package from GitHub before running the analysis) — github.com/ConesaLab/MultiPower
- **FDRsampsize** (Required R dependency of MultiPower for power calculations)
- **lpSolve** (Required R dependency of MultiPower for the cost-constrained optimization of per-omic sample sizes)

## Examples

```
devtools::install_github("ConesaLab/MultiPower"); optResults <- MultiPower(data, groups, type, equalSize=FALSE, cost=...); params <- postMultiPower(optResults); powerPlot(params, params$optimalSampleSize, omicCol)
```

## Evaluation signals

- The postMultiPower trade-off curve reports a detectable Cohen's d of 1.98 at n = 4 samples per condition on the STATegra scenario
- The per-omic optimal sample sizes at equal total cost match 17 (miRNA-seq), 17 (DNase-seq), 9 (RNA-seq) and 6 (metabolomics) per group
- The rendered trade-off figure (postmultipower_tradeoff_curve.png) is produced by powerPlot(parameters, optimalSampleSize, omicCol) with per-omic colors from omicCol
- The MultiPower() call used equalSize=FALSE; if equalSize=TRUE was used the result would be a single common sample size rather than the unequal per-omic optimum
- A results table accompanies the figure listing the per-omic optimal sample sizes from the optimization object

## Limitations

- Input data must be properly preprocessed and batch-effect free before applying MultiPower; the tool itself does not handle preprocessing or batch correction
- MultiPower parameter estimates depend on pilot data availability; without pilot data users must set parameters manually
- Power diminishes as the number of features increases due to multiple testing corrections controlling false positives, which affects the detectable effect size on the curve
- MultiPower is not directly applicable to ML sample-classification sample-size estimation; use MultiML for classification error rate targets instead

## Evidence

- [methods] postMultiPower and powerPlot are the post-hoc analysis functions: "- `postMultiPower(optResults, max.size, omicCol)`"
- [methods] powerPlot renders the trade-off curve from the optimization results: "- `powerPlot(parameters, optimalSampleSize, omicCol)`"
- [methods] MultiPower() supports unequal per-omic sample sizes via equalSize parameter: "- `MultiPower(data, groups, type, omicPower, averagePower, null.effect, fdr, cost, equalSize, max.size, omicCol, powerPlots)`"
- [abstract] MultiPower supports graphical exploration for experimental design decisions: "MultiPower supports different experimental settings, data types and sample sizes, and includes graphical for experimental design decision-making"
- [discussion] Input data preprocessing and batch-effect removal is a prerequisite: "Data should have been properly preprocessed and eventual batch effects, removed."
- [readme] MultiPower is installed as an R package from GitHub: "MultiPower is available as an R package, and can be installed as follows:"
