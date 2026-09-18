---
name: cohen-effect-size-feature-filtering
description: Use when preparing preprocessed, batch-effect-free multi-omic pilot data
  (e.g. STATegra RNA-seq, miRNA-seq, ChIP-seq, DNase-seq, metabolomics, proteomics)
  for MultiPower sample-size estimation and a minimum effect size (Cohen's d) has
  been chosen.
when_to_use_negative:
- The goal is ML sample classification sample-size estimation (MultiML) rather than
  differential-feature power analysis — significance threshold and effect size parameters
  are not applicable there.
- The data still contain batch effects or are not preprocessed; MultiPower/CohenFilter
  require batch-effect-free input.
- No pilot data are available and the user cannot supply parameter estimates manually.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MultiPower
  role: R package whose CohenFilter(data, d, parameters) function performs the per-omic
    Cohen's d feature filtering before power analysis
  repo: github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:bfec2dda685045f05b714b87136d54d060801567f38306a9b0d1059ea19a0411
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/MultiPower
  tools:
  - MultiPower
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cohen-effect-size-feature-filtering

## Summary

Filters features in each omic dataset of a multi-omics pilot study to those showing an effect size of at least Cohen's d (e.g. d = 0.8), using the CohenFilter function of the MultiPower R package, so that downstream power/sample-size calculations (MultiPower, optimalRep) are based on biologically relevant differential features.

## When to use

Use when preparing preprocessed, batch-effect-free multi-omic pilot data (e.g. STATegra RNA-seq, miRNA-seq, ChIP-seq, DNase-seq, metabolomics, proteomics) for MultiPower sample-size estimation and a minimum effect size (Cohen's d) has been chosen.

## When NOT to use

- The goal is ML sample classification sample-size estimation (MultiML) rather than differential-feature power analysis — significance threshold and effect size parameters are not applicable there.
- The data still contain batch effects or are not preprocessed; MultiPower/CohenFilter require batch-effect-free input.
- No pilot data are available and the user cannot supply parameter estimates manually.

## Inputs

- Preprocessed, batch-effect-free multi-omic pilot dataset (list of omic feature matrices, e.g. STATegra RNA-seq, miRNA-seq, ChIP-seq, DNase-seq, metabolomics, proteomics)
- Sample group labels for the pilot samples
- Cohen's d effect-size threshold (e.g. d = 0.8)

## Outputs

- Per-omic feature matrices restricted to features with effect size >= d
- Filtered data object suitable as input to MultiPower/MultiGroupPower and optimalRep

## How to apply

Apply CohenFilter(data, d, parameters) from the MultiPower R package (installed via devtools::install_github('ConesaLab/MultiPower') with FDRsampsize and lpSolve dependencies) to the pilot data loaded as a list of omic matrices with sample group labels, retaining only features whose effect size meets or exceeds the chosen Cohen's d threshold (d = 0.8 in the STATegra workflow). Run the filter per omic before calling MultiPower with fdr = 0.05, omicPower = 0.6, averagePower = 0.8, equalSize = TRUE, then optimalRep and postMultiPower. The rationale is that power diminishes as the number of features increases due to multiple-testing corrections, so restricting features to those with a substantial expected effect yields realistic power and sample-size estimates. Input data must already be properly preprocessed with batch effects removed, since neither CohenFilter nor MultiPower handles this.

## Related tools

- **MultiPower** (R package whose CohenFilter(data, d, parameters) function performs the per-omic Cohen's d feature filtering before power analysis) — github.com/ConesaLab/MultiPower

## Examples

```
CohenFilter(data, d = 0.8)
```

## Evaluation signals

- Each output omic matrix contains only features whose effect size is at least the chosen d threshold (e.g. d = 0.8), so per-omic feature counts are reduced relative to the input.
- The filtered object is accepted by MultiPower without errors when called with fdr = 0.05, omicPower = 0.6, averagePower = 0.8, equalSize = TRUE.
- On the STATegra reference workflow, the full chain (CohenFilter d = 0.8, MultiPower, optimalRep) reproduces the optimal sample size of 16 replicates driven by DNase-seq reaching minimum power 0.6.
- postMultiPower per-omic powers at the optimal size match the published Table 1 values (RNA-seq 0.999, miRNA-seq 0.680, ChIP-seq 0.898, DNase-seq 0.627, Metabolomics 1.000, Proteomics 0.685).
- powerPlot curves for the filtered data show monotone power increase with sample size for every omic at the expected n range.

## Limitations

- Estimates for MultiPower parameters (including the effect size used by CohenFilter) are optimally calculated from pilot data; without pilot data they must be set manually, which can bias the filter.
- Input data must have been properly preprocessed and batch effects removed beforehand; the filter does not handle this.
- The effect-size criterion is only meaningful for differential-features analysis; it is not applicable to ML classification settings where significance threshold and effect size do not apply.

## Evidence

- [methods] CohenFilter(data, d, parameters): "CohenFilter(data, d, parameters)"
- [discussion] Power diminishes as the number of features increases due to the application of multiple testing corrections to control false positives.: "Power diminishes as the number of features increases due to the application of multiple testing corrections to control false positives."
- [discussion] Data should have been properly preprocessed and eventual batch effects, removed.: "Data should have been properly preprocessed and eventual batch effects, removed."
- [readme] Some dependencies are required before running MultiPower that can be installed from R via `install.packages()` and loaded with `library()`: "Some dependencies are required before running MultiPower that can be installed from R via `install.packages()` and loaded with `library()`"
