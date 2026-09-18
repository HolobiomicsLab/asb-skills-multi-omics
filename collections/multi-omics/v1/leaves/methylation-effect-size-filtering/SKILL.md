---
name: methylation-effect-size-filtering
description: Use when a MultiPower run on a multi-omic design with Methyl-seq (or
  another high-feature-count omic) yields an inflated optimal sample size driven by
  low-effect methylation features, and you want to test whether filtering boosts power.
when_to_use_negative:
- Methylation is not the limiting omic in the MultiPower estimate — filtering will
  not change the global sample size set by another platform
- Input data are raw or contain batch effects — MultiPower requires properly preprocessed,
  batch-effect-free data before any filtering is meaningful
- The goal is machine-learning classification sample-size estimation (use MultiML
  instead, where significance threshold and effect size are not applicable)
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MultiPower
  role: R package whose MultiPower(), optimalRep(), postMultiPower() and powerPlot()
    functions re-estimate the optimal multi-omic sample size after methylation feature
    filtering
  repo: https://github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:1ee457f8d126425b53ff4401a9ae70f9ccf132912bd64bf2d8f92d66159f5535
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/MultiPower
  tools:
  - MultiPower
  edam_operation: http://edamontology.org/operation_3233
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# methylation-effect-size-filtering

## Summary

A dimension-reduction step for MultiPower sample-size studies: retain only methylation features with absolute log2 fold change > 0.05 before estimating the optimal sample size, so that high numbers of non-informative methylation features no longer dominate the multi-omic power calculation.

## When to use

Use when a MultiPower run on a multi-omic design with Methyl-seq (or another high-feature-count omic) yields an inflated optimal sample size driven by low-effect methylation features, and you want to test whether filtering boosts power.

## When NOT to use

- Methylation is not the limiting omic in the MultiPower estimate — filtering will not change the global sample size set by another platform
- Input data are raw or contain batch effects — MultiPower requires properly preprocessed, batch-effect-free data before any filtering is meaningful
- The goal is machine-learning classification sample-size estimation (use MultiML instead, where significance threshold and effect size are not applicable)

## Inputs

- Preprocessed, batch-effect-free methylation data matrix (Methyl-seq features x samples) from a multi-omic pilot study
- Per-feature log2 fold-change estimates between biological groups
- Sample-to-group assignments for the multi-omic design

## Outputs

- Filtered methylation data matrix retaining only sites with |log2FC| > 0.05
- Re-computed MultiPower parameters on the filtered data
- Per-omic and global optimal sample sizes (e.g., gbm_multipower_sample_size.csv)
- Power curves from powerPlot(parameters, optimalSampleSize, omicCol)

## How to apply

After a complete-design MultiPower run identifies the limiting omic (in the TCGA Glioblastoma case, Methyl-seq with high feature numbers set n=24), filter the methylation data matrix to keep only sites whose absolute log2 fold change between biological groups exceeds 0.05. Re-estimate MultiPower parameters from the filtered pilot data and re-run MultiPower(data, groups, type, omicPower, averagePower, null.effect, fdr, cost, equalSize, max.size, omicCol, powerPlots) together with optimalRep() and postMultiPower(). The rationale is that power diminishes as feature number increases due to multiple-testing corrections, so removing low-effect-size sites raises per-omic power and lowers the required sample size (24 to 22 for TCGA Glioblastoma). Ensure the input methylation data are preprocessed and batch-effect free before filtering, since MultiPower does not handle preprocessing internally.

## Related tools

- **MultiPower** (R package whose MultiPower(), optimalRep(), postMultiPower() and powerPlot() functions re-estimate the optimal multi-omic sample size after methylation feature filtering) — https://github.com/ConesaLab/MultiPower

## Examples

```
meth_filtered <- meth[abs(meth$log2FC) > 0.05, ]; results <- MultiPower(data, groups, type, omicPower, averagePower, null.effect, fdr, cost, equalSize, max.size, omicCol, powerPlots); optimalRep(results)
```

## Evaluation signals

- Complete four-platform TCGA Glioblastoma design reproduces an optimal sample size of n=24 with Methyl-seq as the limiting platform
- After keeping only methylation sites with |log2FC| > 0.05, the optimal sample size drops from 24 to 22
- All methylation features retained in the filtered matrix satisfy the |log2FC| > 0.05 threshold
- Per-omic and average optimal sample sizes are written to gbm_multipower_sample_size.csv and are consistent with optimalRep()/postMultiPower() output
- Power curves generated by powerPlot() show increased methylation power at fixed sample sizes relative to the unfiltered run

## Limitations

- MultiPower parameter estimates depend on pilot data availability; otherwise users must set parameters manually
- Input data must be preprocessed and batch-effect free before applying MultiPower; the tool provides no internal handling
- The 0.05 |log2FC| cutoff is a study-specific choice from the TCGA Glioblastoma task, not a universally validated threshold

## Evidence

- [results] Filtering reduces required sample size: [evidence span withheld — anchor status: not_found]
- [discussion] Feature-count rationale: "Power diminishes as the number of features increases due to the application of multiple testing corrections to control false positives."
- [readme] Tool installation: "devtools::install_github(“ConesaLab/MultiPower”)"
- [discussion] Preprocessing prerequisite: "Data should have been properly preprocessed and eventual batch effects, removed."
