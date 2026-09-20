---
name: r-package-statistical-workflow
description: Use when designing a multi-omics experiment and you need per-omic optimal
  sample sizes (optionally unconstrained, unequal sizes at equal cost) plus a sample-size
  vs detectable Cohen's d trade-off curve from pilot omics matrices in R.
when_to_use_negative:
- Input data still contain batch effects or lack proper preprocessing — MultiPower
  requires preprocessed, batch-effect-free pilot data and does not handle this itself
- The goal is sample-size estimation for machine-learning sample classification —
  MultiPower's significance-threshold/effect-size framework is not applicable; use
  MultiML instead
- No pilot data are available and parameters cannot be set manually — parameter estimates
  depend on pilot data availability
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_2269
tools:
- name: MultiPower
  role: 'Core R package: estimates optimal sample size per omic and supports unequal
    per-omic sizes at equal cost via MultiPower()/optimalRep(), with postMultiPower()/powerPlot()
    for the trade-off curve'
  repo: github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:9f2d18e17053e8f133c823c0316340b4f4108b6f396339c8d2efc517088cc448
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiPower
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# r-package-statistical-workflow

## Summary

End-to-end R workflow for running the MultiPower sample-size/power pipeline on preprocessed multi-omics pilot data, from GitHub installation through optimalRep and postMultiPower trade-off curve rendering, as demonstrated on the STATegra dataset (miRNA-seq, RNA-seq, DNase-seq, metabolomics).

## When to use

Use when designing a multi-omics experiment and you need per-omic optimal sample sizes (optionally unconstrained, unequal sizes at equal cost) plus a sample-size vs detectable Cohen's d trade-off curve from pilot omics matrices in R.

## When NOT to use

- Input data still contain batch effects or lack proper preprocessing — MultiPower requires preprocessed, batch-effect-free pilot data and does not handle this itself
- The goal is sample-size estimation for machine-learning sample classification — MultiPower's significance-threshold/effect-size framework is not applicable; use MultiML instead
- No pilot data are available and parameters cannot be set manually — parameter estimates depend on pilot data availability

## Inputs

- Preprocessed, batch-effect-free pilot multi-omics data matrices (e.g. miRNA-seq, RNA-seq, DNase-seq, metabolomics) loaded in R
- Group/condition labels per sample (groups argument)
- Omic data type annotations (type argument)
- Cost constraint and target FDR / power parameters (cost, fdr, omicPower, averagePower)
- equalSize flag and max.size bound for the sample-size optimization

## Outputs

- Per-omic optimal sample sizes at equal cost (e.g. 17/17/9/6 replicates per group for STATegra miRNA-seq/DNase-seq/RNA-seq/metabolomics)
- Sample-size vs detectable effect-size (Cohen's d) trade-off object from postMultiPower
- Trade-off curve rendered via powerPlot (e.g. written to postmultipower_tradeoff_curve.png)
- Results table of per-omic optimal sample sizes

## How to apply

Install the package from GitHub via devtools::install_github("ConesaLab/MultiPower") after installing required dependencies FDRsampsize and lpSolve with install.packages(). Load preprocessed, batch-effect-free pilot matrices for each omic (e.g. miRNA-seq, RNA-seq, DNase-seq, metabolomics) and call MultiPower(data, groups, type, omicPower, averagePower, null.effect, fdr, cost, equalSize, max.size, omicCol, powerPlots); set equalSize=FALSE with a cost constraint to allow different per-omic replicate numbers at equal total cost. Feed the optimization object to postMultiPower(optResults, max.size, omicCol) to compute the sample-size vs detectable effect-size trade-off, and render it with powerPlot(parameters, optimalSampleSize, omicCol). On STATegra with equalSize=FALSE at equal cost, the expected solution admits 17 replicates per group for miRNA-seq and DNase-seq, 9 for RNA-seq, and 6 for metabolomics, with a detectable Cohen's d of 1.98 at n=4 per condition.

## Related tools

- **MultiPower** (Core R package: estimates optimal sample size per omic and supports unequal per-omic sizes at equal cost via MultiPower()/optimalRep(), with postMultiPower()/powerPlot() for the trade-off curve) — github.com/ConesaLab/MultiPower

## Examples

```
devtools::install_github("ConesaLab/MultiPower"); optResults <- MultiPower(data, groups, type, equalSize=FALSE, cost=costVec, powerPlots=TRUE); post <- postMultiPower(optResults); powerPlot(optResults$parameters, post$optimalSampleSize, omicCol)
```

## Evaluation signals

- postMultiPower trade-off curve reports a detectable Cohen's d of 1.98 at n=4 samples per condition on the STATegra scenario
- Optimal per-omic sample sizes at equal cost match 17 (miRNA-seq), 17 (DNase-seq), 9 (RNA-seq), and 6 (metabolomics) replicates per group with equalSize=FALSE
- Package installs cleanly after FDRsampsize and lpSolve dependencies are present (devtools::install_github("ConesaLab/MultiPower") succeeds)
- powerPlot produces a rendered trade-off curve file (e.g. postmultipower_tradeoff_curve.png) alongside a per-omic optimal-sample-size results table
- Optimization respects the cost constraint and max.size bound while allowing unequal per-omic sizes

## Limitations

- Input data must be properly preprocessed with batch effects removed beforehand; MultiPower provides no handling for this
- MultiPower parameter estimates are optimally calculated from pilot data; without pilot data users must set parameters manually
- MultiPower is not directly applicable to ML sample-classification methods because significance threshold and effect size do not apply there (use MultiML, which also requires pilot data)

## Evidence

- [methods] Install MultiPower as an R package from GitHub: "devtools::install_github(“ConesaLab/MultiPower”)"
- [methods] MultiPower function signature with equalSize and cost parameters: "MultiPower(data, groups, type, omicPower, averagePower, null.effect, fdr, cost, equalSize, max.size, omicCol, powerPlots)"
- [methods] Post-processing and plotting functions for the trade-off curve: "postMultiPower(optResults, max.size, omicCol)"
- [readme] Dependencies required before running MultiPower: "Some dependencies are required before running MultiPower that can be installed from R via install.packages() and loaded with library(): - FDRsampsize - lpSolve"
- [discussion] Pilot data requirement for parameter estimation: "Estimates for MultiPower parameters are optimally calculated from pilot data, although they can also be manually provided."
- [discussion] Preprocessing and batch-effect prerequisite: "Data should have been properly preprocessed and eventual batch effects, removed."
