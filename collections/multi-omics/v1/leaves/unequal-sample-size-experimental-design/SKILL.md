---
name: unequal-sample-size-experimental-design
description: Use when designing a multi-omics experiment where the equal-replicate
  constraint is too restrictive and you want MultiPower to redistribute replicates
  per omic (e.g. RNA-seq, miRNA-seq, DNase-seq, metabolomics) at equal total cost.
when_to_use_negative:
- When the design requires the same sample size across all platforms — use equalSize=TRUE
  instead (which on STATegra gives a common n=16 driven by DNase-seq)
- When the goal is sample-size estimation for machine learning classification error
  rates rather than differential feature power — MultiML is the complementary method
  for that setting
- When input data have unremoved batch effects or are not preprocessed — MultiPower
  assumes pilot data that have been properly preprocessed with batch effects removed
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3912
tools:
- name: MultiPower
  role: R package whose MultiPower()/optimalRep()/postMultiPower()/powerPlot() functions
    compute per-omic power, optimal unequal sample sizes under a cost constraint,
    and the trade-off curve
  repo: github.com/ConesaLab/MultiPower
- name: devtools
  role: Used to install the MultiPower package from GitHub via devtools::install_github("ConesaLab/MultiPower")
- name: lpSolve
  role: CRAN dependency used by MultiPower's optimizer to solve the equal-cost sample-size
    allocation across omics
- name: FDRsampsize
  role: CRAN dependency for the false-discovery-rate-aware power calculations underlying
    MultiPower
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:4d62939a5365c9b2189c81ce8a257f32e8b643b2c70185271503b6f03ce9257e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FDRsampsize
  - MultiPower
  - devtools
  - lpSolve
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3912
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# unequal-sample-size-experimental-design

## Summary

Use MultiPower with equalSize=FALSE and a total-cost constraint to let each omic platform receive a different number of replicates at equal cost, optimizing per-omic sample sizes and power for multi-omics experimental design.

## When to use

Use when designing a multi-omics experiment where the equal-replicate constraint is too restrictive and you want MultiPower to redistribute replicates per omic (e.g. RNA-seq, miRNA-seq, DNase-seq, metabolomics) at equal total cost.

## When NOT to use

- When the design requires the same sample size across all platforms — use equalSize=TRUE instead (which on STATegra gives a common n=16 driven by DNase-seq)
- When the goal is sample-size estimation for machine learning classification error rates rather than differential feature power — MultiML is the complementary method for that setting
- When input data have unremoved batch effects or are not preprocessed — MultiPower assumes pilot data that have been properly preprocessed with batch effects removed

## Inputs

- Preprocessed, batch-effect-free pilot multi-omics data matrices (e.g. RNA-seq, miRNA-seq, ChIP-seq, DNase-seq, metabolomics, proteomics) with sample group labels
- Group assignment vector for samples
- Omic type specification (e.g. RNAseq, counts vs continuous)
- Cost vector per sample per omic and power parameters (fdr, omicPower, averagePower, null.effect, max.size)

## Outputs

- Per-omic optimal sample sizes at equal total cost (e.g. 17/17/9/6 on STATegra)
- Sample-size vs detectable Cohen's d trade-off table from postMultiPower
- Power/trade-off curve rendered via powerPlot (e.g. postmultipower_tradeoff_curve.png)
- Per-omic power estimates at the chosen sizes

## How to apply

Load preprocessed, batch-effect-free pilot multi-omics matrices (e.g. miRNA-seq, RNA-seq, DNase-seq, metabolomics) into R after installing the package via devtools::install_github from ConesaLab/MultiPower, with FDRsampsize and lpSolve from CRAN as dependencies. Call MultiPower(data, groups, type, omicPower, averagePower, null.effect, fdr, cost, equalSize, max.size, omicCol, powerPlots) with equalSize=FALSE and a cost constraint, so the optimizer (lpSolve-based) can admit different sample sizes per omic at equal total cost; on STATegra this yields n=17 for miRNA-seq and DNase-seq, n=9 for RNA-seq, and n=6 for metabolomics. Feed the optimization object to postMultiPower(optResults, max.size, omicCol) to compute the sample-size versus detectable effect-size (Cohen's d) trade-off, and render it with powerPlot(parameters, optimalSampleSize, omicCol). Accept the result if the trade-off curve shows a detectable Cohen's d of 1.98 at n=4 per condition and the per-omic optima match the equal-cost solution (17/17/9/6).

## Related tools

- **MultiPower** (R package whose MultiPower()/optimalRep()/postMultiPower()/powerPlot() functions compute per-omic power, optimal unequal sample sizes under a cost constraint, and the trade-off curve) — github.com/ConesaLab/MultiPower
- **devtools** (Used to install the MultiPower package from GitHub via devtools::install_github("ConesaLab/MultiPower"))
- **lpSolve** (CRAN dependency used by MultiPower's optimizer to solve the equal-cost sample-size allocation across omics)
- **FDRsampsize** (CRAN dependency for the false-discovery-rate-aware power calculations underlying MultiPower)

## Examples

```
MultiPower(data, groups, type, fdr = 0.05, omicPower = 0.6, averagePower = 0.8, cost = cost, equalSize = FALSE)
```

## Evaluation signals

- The per-omic optimal sizes satisfy the total-cost constraint while differing across omics; on STATegra at equal cost they must be 17 (miRNA-seq), 17 (DNase-seq), 9 (RNA-seq), 6 (metabolomics)
- The postMultiPower trade-off curve reports a detectable Cohen's d of 1.98 at n=4 samples per condition
- Each omic's power at its allocated size meets the target minimum omicPower (e.g. 0.6) while average power reaches the target (e.g. 0.8) at FDR 0.05
- All MultiPower dependencies (FDRsampsize, lpSolve) load without error and devtools::install_github("ConesaLab/MultiPower") completes
- powerPlot renders a monotone sample-size–detectable-effect curve consistent with the power diminishing as feature count and multiple-testing correction increase

## Limitations

- Input data must be properly preprocessed and batch-effect free before applying MultiPower; the tool performs no such handling
- MultiPower parameter estimates are optimally calculated from pilot data; without pilot data users must set parameters manually
- Power diminishes as the number of features increases due to multiple testing corrections, so results are sensitive to the feature-filtering step (e.g. CohenFilter with d=0.8)
- The optimization is bounded by max.size and the cost constraint; relaxing equalSize only helps if the cost model permits redistributing replicates

## Evidence

- [methods] MultiPower call with equalSize=FALSE: "MultiPower(data, groups, type, omicPower, averagePower, null.effect, fdr, cost, equalSize, max.size, omicCol, powerPlots)"
- [abstract] MultiPower breadth of supported settings: "MultiPower supports different experimental settings, data types and sample sizes, and includes graphical for experimental design decision-making"
- [discussion] Preprocessing prerequisite: "Data should have been properly preprocessed and eventual batch effects, removed."
- [readme] Installation route: "devtools::install_github(“ConesaLab/MultiPower”)"
