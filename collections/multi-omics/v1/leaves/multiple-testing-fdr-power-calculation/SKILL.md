---
name: multiple-testing-fdr-power-calculation
description: Use when designing or evaluating a multi-omic differential-features study
  and you need a per-platform replicate number that meets target per-omic and average
  power under an FDR threshold, e.g. omicPower 0.6, averagePower 0.8, FDR 0.05.
when_to_use_negative:
- The goal is sample-size estimation for machine learning sample classification rather
  than differential features analysis — significance threshold and effect size are
  not applicable, so use MultiML instead
- No pilot data are available and parameter estimates cannot be computed — MultiPower
  estimates are optimally calculated from pilot data, otherwise parameters must be
  set manually
- Input data have not been preprocessed or batch effects have not been removed — MultiPower
  does not handle these internally
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3518
tools:
- name: MultiPower
  role: R package that performs statistical power studies for multi-omics experiments;
    CohenFilter, optimalRep, postMultiPower and powerPlot drive the sample-size estimation
  repo: github.com/ConesaLab/MultiPower
- name: FDRsampsize
  role: CRAN dependency required before running MultiPower
- name: lpSolve
  role: CRAN dependency required before running MultiPower
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:5bcb12b771355b410514fe0f14f23de331c2e0b78cbf523cd1083740f45b86b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FDRsampsize
  - MultiPower
  - lpSolve
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# multiple-testing-fdr-power-calculation

## Summary

Estimates the optimal number of replicates per omic platform in a multi-omics experiment using the MultiPower R package, which computes statistical power under multiple-testing (FDR) correction across omics such as RNA-seq, miRNA-seq, ChIP-seq, DNase-seq, metabolomics and proteomics.

## When to use

Use when designing or evaluating a multi-omic differential-features study and you need a per-platform replicate number that meets target per-omic and average power under an FDR threshold, e.g. omicPower 0.6, averagePower 0.8, FDR 0.05.

## When NOT to use

- The goal is sample-size estimation for machine learning sample classification rather than differential features analysis — significance threshold and effect size are not applicable, so use MultiML instead
- No pilot data are available and parameter estimates cannot be computed — MultiPower estimates are optimally calculated from pilot data, otherwise parameters must be set manually
- Input data have not been preprocessed or batch effects have not been removed — MultiPower does not handle these internally

## Inputs

- Preprocessed, batch-effect-free pilot multi-omics data matrix (e.g. STATegra RNA-seq, miRNA-seq, ChIP-seq, DNase-seq, metabolomics, proteomics features x samples)
- Sample group labels for differential comparison
- Power parameters: fdr (0.05), omicPower (0.6), averagePower (0.8), Cohen's d (0.8), equalSize flag

## Outputs

- Optimal common sample size across platforms (e.g. n = 16 for STATegra)
- Per-omic power table at the optimal sample size (e.g. RNA-seq 0.999, miRNA-seq 0.680, ChIP-seq 0.898, DNase-seq 0.627, metabolomics 1.000, proteomics 0.685)
- Power plots showing sample-size–power relationships per omic

## How to apply

Start from a preprocessed, batch-effect-free pilot multi-omics dataset with sample group labels (e.g. STATegra: RNA-seq, miRNA-seq, ChIP-seq, DNase-seq, metabolomics, proteomics). Install MultiPower from GitHub (devtools::install_github('ConesaLab/MultiPower')) and its CRAN dependencies FDRsampsize and lpSolve. Apply CohenFilter(data, d = 0.8) to retain features with effect size at least Cohen's d = 0.8, then call MultiPower/optimalRep with fdr = 0.05, omicPower = 0.6, averagePower = 0.8 and equalSize = TRUE to enforce a common sample size across platforms. Use postMultiPower (with omicCol) to report per-omic power at the optimal size and powerPlot to visualize the sample-size–power relationship. Rationale: power diminishes as the number of features increases due to multiple testing corrections, so filtering by effect size and setting FDR-controlled power targets drives the estimated replicate number; on STATegra this yields n = 16 replicates set by the DNase-seq platform's power requirement.

## Related tools

- **MultiPower** (R package that performs statistical power studies for multi-omics experiments; CohenFilter, optimalRep, postMultiPower and powerPlot drive the sample-size estimation) — github.com/ConesaLab/MultiPower
- **FDRsampsize** (CRAN dependency required before running MultiPower)
- **lpSolve** (CRAN dependency required before running MultiPower)

## Examples

```
devtools::install_github("ConesaLab/MultiPower"); parameters <- MultiPower(data, groups, type, omicPower = 0.6, averagePower = 0.8, fdr = 0.05, equalSize = TRUE); opt <- optimalRep(parameters, omicPower = 0.6, averagePower = 0.8, fdr = 0.05, equalSize = TRUE)
```

## Evaluation signals

- optimalRep returns a common sample size of 16 replicates for the STATegra data under fdr = 0.05, omicPower = 0.6, averagePower = 0.8, d = 0.8, equalSize = TRUE
- Per-omic powers at the optimal size match Table 1: RNA-seq 0.999, miRNA-seq 0.680, ChIP-seq 0.898, DNase-seq 0.627, Metabolomics 1.000, Proteomics 0.685
- The binding constraint is DNase-seq reaching the minimum per-omic power of 0.6
- CohenFilter retains only features with effect size >= 0.8 per omic before power estimation
- Resulting power plots show per-omic power increasing monotonically with sample size up to the optimum

## Limitations

- MultiPower is not directly applicable to ML methods used for sample classification, as the basic parameters of the power calculation (significance threshold and effect size) are not applicable
- Estimates for MultiPower parameters are optimally calculated from pilot data, although they can also be manually provided
- Input data should have been properly preprocessed and eventual batch effects removed; the tool does not do this itself
- Power diminishes as the number of features increases due to multiple testing corrections controlling false positives

## Evidence

- [abstract] MultiPower estimates and assesses the optimal sample size in a multi-omics experiment: "we formulate the MultiPower method to estimate and assess the optimal sample size in a multi-omics experiment"
- [readme] Installation from GitHub: "devtools::install_github(“ConesaLab/MultiPower”)"
- [readme] Dependencies from CRAN: "Some dependencies are required before running MultiPower that can be installed from R via `install.packages()` and loaded with `library()`:

- FDRsampsize
- lpSolve"
- [methods] Function signatures: "- `optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect)`"
- [discussion] Power decreases with feature count: "Power diminishes as the number of features increases due to the application of multiple testing corrections to control false positives."
- [discussion] Pilot data requirement: "Estimates for MultiPower parameters are optimally calculated from pilot data, although they can also be manually provided."
