---
name: tcga-cohort-data-handling
description: Use when pilot/cohort data come from TCGA or a similar multi-omic cohort
  and the goal is to estimate required sample size per omic platform (e.g., Methyl-seq
  limiting n=24) or to test whether filtering low-variance features reduces it.
when_to_use_negative:
- Data have not been preprocessed or batch effects have not been removed — MultiPower
  requires input data that are 'properly preprocessed and eventual batch effects,
  removed'
- No pilot data are available and parameters cannot be estimated — MultiPower parameter
  estimates are optimally calculated from pilot data, otherwise they must be set manually
- The goal is sample size for machine-learning classification rather than differential
  feature analysis — use MultiML instead, since MultiPower's significance threshold
  and effect size parameters are not applicable to ML classifiers
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0622
tools:
- name: MultiPower
  role: Core R package for power/sample-size estimation on the TCGA multi-omic cohort;
    run via MultiPower(), optimalRep(), postMultiPower(), powerPlot()
  repo: github.com/ConesaLab/MultiPower
- name: devtools
  role: Used to install the MultiPower package from GitHub
  repo: https://github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:cec222a7a80fb981cfb449bac06bff956ff6576945b1f44e947a7dcc05999bb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/MultiPower
  tools:
  - MultiPower
  - devtools
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# tcga-cohort-data-handling

## Summary

Prepare and analyze TCGA multi-omic cohort data (e.g., four TCGA Glioblastoma platforms including Methyl-seq) as MultiPower input, so that optimal sample sizes can be estimated for complete and feature-filtered multi-omic designs.

## When to use

Use when pilot/cohort data come from TCGA or a similar multi-omic cohort and the goal is to estimate required sample size per omic platform (e.g., Methyl-seq limiting n=24) or to test whether filtering low-variance features reduces it.

## When NOT to use

- Data have not been preprocessed or batch effects have not been removed — MultiPower requires input data that are 'properly preprocessed and eventual batch effects, removed'
- No pilot data are available and parameters cannot be estimated — MultiPower parameter estimates are optimally calculated from pilot data, otherwise they must be set manually
- The goal is sample size for machine-learning classification rather than differential feature analysis — use MultiML instead, since MultiPower's significance threshold and effect size parameters are not applicable to ML classifiers

## Inputs

- Preprocessed, batch-effect-free multi-omic pilot/cohort data matrix from TCGA (e.g., four Glioblastoma omic platforms including Methyl-seq)
- Sample-to-biological-group assignment (groups)
- Omic data type specification (type argument for MultiPower)

## Outputs

- Per-omic and globally optimal sample size estimates (e.g., n=24 complete; n=22 methylation-filtered)
- Power curves from powerPlot(parameters, optimalSampleSize, omicCol)
- Summary CSV of per-omic and average optimal sample sizes (e.g., gbm_multipower_sample_size.csv)

## How to apply

Load preprocessed, batch-effect-free TCGA multi-omic pilot data (e.g., the four TCGA Glioblastoma omic platforms) and assign samples to their biological groups. Run MultiPower(data, groups, type, omicPower, averagePower, null.effect, fdr, cost, equalSize, max.size, omicCol, powerPlots) on the complete design, then use optimalRep() and postMultiPower() to obtain per-omic and global optimal sample sizes. Because power diminishes as the number of features increases due to multiple-testing corrections, filter the platform with the most features (Methyl-seq) — e.g., retain only methylation sites with absolute log2 fold change > 0.05 — and re-run MultiPower to confirm the sample size drops (24 to 22 in the TCGA GBM case). Generate power curves with powerPlot() and write per-omic and average optimal sample sizes to a CSV such as gbm_multipower_sample_size.csv.

## Related tools

- **MultiPower** (Core R package for power/sample-size estimation on the TCGA multi-omic cohort; run via MultiPower(), optimalRep(), postMultiPower(), powerPlot()) — github.com/ConesaLab/MultiPower
- **devtools** (Used to install the MultiPower package from GitHub) — https://github.com/ConesaLab/MultiPower

## Examples

```
devtools::install_github("ConesaLab/MultiPower"); res <- MultiPower(data, groups, type, fdr=0.05, powerPlots=TRUE); opt <- optimalRep(res)
```

## Evaluation signals

- Complete-design optimal sample size equals 24, with Methyl-seq identified as the limiting platform due to its high number of features
- After filtering methylation sites to absolute log2 fold change > 0.05, the optimal sample size decreases from 24 to 22
- optimalRep() and postMultiPower() return consistent per-omic and global sample sizes across the complete and filtered runs
- Output CSV (e.g., gbm_multipower_sample_size.csv) contains both per-omic and average optimal sample sizes
- powerPlot() power curves render for all omics in omicCol without errors

## Limitations

- Input data must be properly preprocessed and batch effects removed beforehand; MultiPower does not handle this internally
- Parameter estimates are optimally calculated from pilot data; without pilot data users must provide parameters manually
- Power diminishes as the number of features increases because of multiple testing corrections, so feature-rich platforms like Methyl-seq dominate the required sample size

## Evidence

- [intro] TCGA cohort multi-omic design: "Human multi-omic cohort studies such as The Cancer Genome Atlas database (TCGA) usually collect data from a large number of subjects"
- [discussion] Power decreases with feature number: "Power diminishes as the number of features increases due to the application of multiple testing corrections to control false positives."
- [discussion] Preprocessing requirement: "Data should have been properly preprocessed and eventual batch effects, removed."
- [readme] Install from GitHub: "devtools::install_github(“ConesaLab/MultiPower”)"
- [methods] MultiPower function signature: "MultiPower(data, groups, type, omicPower, averagePower, null.effect, fdr, cost, equalSize, max.size, omicCol, powerPlots)"
