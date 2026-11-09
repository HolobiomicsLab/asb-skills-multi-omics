---
name: multi-omics-power-analysis
description: Use when designing a multi-omic experiment (e.g. RNA-seq, miRNA-seq,
  DNase-seq, metabolomics) and you need the optimal number of replicates per omic
  to reach target per-omic and average power at a given FDR and Cohen's d.
when_to_use_negative:
- The goal is sample-size estimation for machine-learning classification of samples
  — use MultiML instead, since significance threshold and effect size are not applicable
  to ML power calculations
- Data still contain batch effects or are not properly preprocessed — MultiPower does
  not handle this internally and requires clean pilot data
- No pilot data are available and parameters cannot be estimated — parameter estimates
  depend on pilot data, otherwise users must set them manually
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0092
tools:
- name: MultiPower
  role: 'Core R package: CohenFilter, MultiPower, optimalRep, postMultiPower, and
    powerPlot implement power estimation and optimal sample-size calculation across
    omics'
  repo: github.com/ConesaLab/MultiPower
- name: lpSolve
  role: CRAN dependency providing the mixed-integer linear programming solver behind
    optimalRep's constrained sample-size optimization
- name: FDRsampsize
  role: CRAN dependency required before running MultiPower, used for FDR-based power/sample-size
    computations
- name: MultiML
  role: Complementary method for ML classification sample-size estimation; not the
    path for differential-analysis power, so use only when the goal is classification
    error-rate control
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_003
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:bd4e2c5800681fe2806c596ee8f05a8415deb511de123873296752b710afa0c9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FDRsampsize
  - MultiML
  - MultiPower
  - lpSolve
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# multi-omics-power-analysis

## Summary

Statistical power analysis and optimal sample-size estimation for multi-omics differential-analysis experiments using the MultiPower R package, which models per-feature variability, effect size, and number of features across omic platforms under a constrained optimization.

## When to use

Use when designing a multi-omic experiment (e.g. RNA-seq, miRNA-seq, DNase-seq, metabolomics) and you need the optimal number of replicates per omic to reach target per-omic and average power at a given FDR and Cohen's d.

## When NOT to use

- The goal is sample-size estimation for machine-learning classification of samples — use MultiML instead, since significance threshold and effect size are not applicable to ML power calculations
- Data still contain batch effects or are not properly preprocessed — MultiPower does not handle this internally and requires clean pilot data
- No pilot data are available and parameters cannot be estimated — parameter estimates depend on pilot data, otherwise users must set them manually

## Inputs

- Preprocessed, batch-effect-free pilot multi-omics matrices (e.g. RNA-seq, miRNA-seq, ChIP-seq, DNase-seq, metabolomics, proteomics count matrices) with sample group labels
- Cohen's d threshold for the pseudo-DE feature set (e.g. d = 0.8)
- Design parameters: fdr (0.05), omicPower (0.6), averagePower (0.8), equalSize flag, optional per-omic cost vector, max.size bound

## Outputs

- Per-omic and global optimal sample sizes (e.g. STATegra n=16; STATegra unequal-size 17/17/9/6; TCGA GBM n=24 or 22 filtered)
- Per-omic power table at the optimal sample size
- Power/trade-off curve plots (powerPlot, e.g. postmultipower_tradeoff_curve.png) and results tables such as multipower_results.csv or optimal_sample_sizes.csv

## How to apply

Install the package from GitHub with devtools::install_github('ConesaLab/MultiPower') plus the FDRsampsize and lpSolve CRAN dependencies, and start from preprocessed, batch-effect-free pilot data per omic. Apply CohenFilter(data, d = 0.8) to flag the pseudo-DE feature set M1 and summarize per-feature pooled standard deviation, variance, mean counts, dispersion, and fold-change as percentiles (Pk, P100-k) into a parameters object. Run MultiPower(data, groups, type, omicPower, averagePower, null.effect, fdr, cost, equalSize, max.size, omicCol, powerPlots) with typical settings fdr = 0.05, omicPower = 0.6, averagePower = 0.8, then call optimalRep() — which solves a 0-1 linear integer program (via lpSolve) minimizing cost subject to per-omic and average power constraints — followed by postMultiPower() and powerPlot() to inspect the sample-size vs detectable-effect trade-off. Set equalSize = TRUE for a common sample size across platforms, or FALSE with a cost constraint to allow different per-omic sizes at equal cost. Validate results by checking that all power constraints hold and by comparing against known benchmark results (e.g. STATegra n=16 equal-size; TCGA GBM n=24 reduced to 22 after methylation filtering).

## Related tools

- **MultiPower** (Core R package: CohenFilter, MultiPower, optimalRep, postMultiPower, and powerPlot implement power estimation and optimal sample-size calculation across omics) — github.com/ConesaLab/MultiPower
- **lpSolve** (CRAN dependency providing the mixed-integer linear programming solver behind optimalRep's constrained sample-size optimization)
- **FDRsampsize** (CRAN dependency required before running MultiPower, used for FDR-based power/sample-size computations)
- **MultiML** (Complementary method for ML classification sample-size estimation; not the path for differential-analysis power, so use only when the goal is classification error-rate control)

## Examples

```
devtools::install_github("ConesaLab/MultiPower"); opt <- optimalRep(parameters, omicPower=0.6, averagePower=0.8, fdr=0.05, equalSize=TRUE)
```

## Evaluation signals

- On STATegra with fdr=0.05, omicPower=0.6, averagePower=0.8, Cohen's d=0.8 and equalSize=TRUE, the optimal common sample size must equal 16 with per-omic powers matching Table 1 (RNA-seq 0.999, miRNA-seq 0.680, ChIP-seq 0.898, DNase-seq 0.627, Metabolomics 1.000, Proteomics 0.685)
- With equalSize=FALSE at equal cost, per-omic sizes must be 17 (miRNA-seq), 17 (DNase-seq), 9 (RNA-seq), 6 (metabolomics), and the postMultiPower trade-off curve must report detectable Cohen's d of 1.98 at n=4
- MILP solution from lpSolve must match optimalRep's reference solution and satisfy all constraints: per-omic power ≥ omicPower, average power ≥ averagePower, n ≤ max.size
- Every parameter field in the CohenFilter output (SD, variance, mean counts, dispersion, fold-change percentiles) must be present and finite before downstream optimalRep() is called
- On TCGA Glioblastoma, the complete four-platform design must yield n=24 (Methyl-seq limiting) and filtering methylation sites to |log2FC| > 0.05 must reduce it to n=22

## Limitations

- Input data must already be preprocessed and batch-effect free; MultiPower does not handle preprocessing or batch correction internally
- MultiPower parameter estimates depend on pilot data availability; without pilot data, users must set parameters manually
- Power diminishes as the number of features increases due to multiple testing corrections — feature filtering (e.g. methylation |log2FC| > 0.05) can reduce required sample size
- MultiPower is not directly applicable to ML sample-classification methods because significance threshold and effect size are not applicable in that setting

## Evidence

- [abstract] MultiPower estimates and assesses the optimal sample size in a multi-omics experiment, supporting different experimental settings, data types and sample sizes: "MultiPower supports different experimental settings, data types and sample sizes, and includes graphical for experimental design decision-making"
- [methods] Install from GitHub: "devtools::install_github(“ConesaLab/MultiPower”)"
- [results] optimization formulation: [evidence span withheld — anchor status: not_found]
- [discussion] pilot data requirement: "Estimates for MultiPower parameters are optimally calculated from pilot data, although they can also be manually provided."
- [readme] dependencies: "Some dependencies are required before running MultiPower that can be installed from R via 
`install.packages()` and loaded with `library()`:

- FDRsampsize
- lpSolve"
