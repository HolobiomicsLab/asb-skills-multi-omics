---
name: correlation-significance-testing
description: Use when you have paired per-method score vectors (e.g. accuracy scores
  from the 49-method simulation benchmark) computed separately on SRT and scRNA-seq
  reference datasets, and you need to test their linear agreement.
when_to_use_negative:
- Only one modality's scores are available (e.g. scores exist solely on scRNA-seq
  reference datasets), so no paired SRT/scRNA-seq vectors can be formed.
- The goal is comparing score distributions or detecting outliers rather than testing
  linear association between paired per-method scores.
- Scores are non-numeric or rank-only and monotonic-but-nonlinear agreement is of
  interest, where Pearson's r is inappropriate.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SRTsim
  role: Simulation method whose accuracy score on SRT reference datasets forms one
    vector of the paired correlation
- name: scDesign3
  role: Simulation method contributing accuracy scores on both SRT and scRNA-seq reference
    datasets to the paired correlation
- name: ZINB-WaVE
  role: Simulation method contributing accuracy scores on both SRT and scRNA-seq reference
    datasets to the paired correlation
- name: scDesign2
  role: Simulation method contributing accuracy scores on both SRT and scRNA-seq reference
    datasets to the paired correlation
- name: simpipe
  role: Standard pipeline used to estimate parameters from real datasets, simulate
    data, and evaluate simulated data, producing the per-method accuracy scores
  repo: https://github.com/duohongrui/simpipe
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:e6180f796a6b7970a5316fe40befe5f9fba343f85a07ff080d00fcf17b6ef353
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simpipe
  tools:
  - SRTsim
  - ZINB-WaVE
  - scDesign2
  - scDesign3
  - simpipe
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# correlation-significance-testing

## Summary

Computes and significance-tests the Pearson correlation between per-method accuracy scores evaluated on SRT reference datasets versus scRNA-seq reference datasets, quantifying how well benchmark rankings transfer across transcriptomics modalities.

## When to use

Use when you have paired per-method score vectors (e.g. accuracy scores from the 49-method simulation benchmark) computed separately on SRT and scRNA-seq reference datasets, and you need to test their linear agreement.

## When NOT to use

- Only one modality's scores are available (e.g. scores exist solely on scRNA-seq reference datasets), so no paired SRT/scRNA-seq vectors can be formed.
- The goal is comparing score distributions or detecting outliers rather than testing linear association between paired per-method scores.
- Scores are non-numeric or rank-only and monotonic-but-nonlinear agreement is of interest, where Pearson's r is inappropriate.

## Inputs

- accuracy_scores.csv with per-method accuracy scores computed on SRT reference datasets
- accuracy_scores.csv with per-method accuracy scores computed on scRNA-seq reference datasets (paired column in same file)
- Method identifier column shared by both score columns

## Outputs

- correlation_srt_scrnaseq record containing pearson_r, p-value, and number of paired methods
- Verification flag that pearson_r reproduces the reported value of ~0.92

## How to apply

Load the accuracy_scores.csv produced by the benchmarking pipeline (task_001) containing per-method accuracy scores computed separately on SRT and scRNA-seq reference datasets. Extract the two paired score columns and drop methods that have missing values in either modality so every observation is paired across the 49 evaluated simulation methods (e.g. SRTsim, scDesign3, ZINB-WaVE, scDesign2). Compute Pearson's r and its p-value with scipy.stats.pearsonr (or R cor.test), and record the number of paired methods contributing to the correlation. Verify that pearson_r reproduces the reported value of approximately 0.92 (Fig. 4b). The rationale is that a strong correlation demonstrates that scRNA-seq-tailored simulators remain compatible for SRT evaluation, so separate modality-specific benchmarks are largely redundant for ranking method accuracy.

## Related tools

- **SRTsim** (Simulation method whose accuracy score on SRT reference datasets forms one vector of the paired correlation)
- **scDesign3** (Simulation method contributing accuracy scores on both SRT and scRNA-seq reference datasets to the paired correlation)
- **ZINB-WaVE** (Simulation method contributing accuracy scores on both SRT and scRNA-seq reference datasets to the paired correlation)
- **scDesign2** (Simulation method contributing accuracy scores on both SRT and scRNA-seq reference datasets to the paired correlation)
- **simpipe** (Standard pipeline used to estimate parameters from real datasets, simulate data, and evaluate simulated data, producing the per-method accuracy scores) — https://github.com/duohongrui/simpipe

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(srt_scores, scrnaseq_scores)
```

## Evaluation signals

- pearson_r falls in [-1, 1] and matches the reported value of approximately 0.92 from Fig. 4b
- A p-value accompanies pearson_r from the same statistical test (scipy.stats.pearsonr)
- The method count equals the number of simulation methods with non-missing scores in BOTH the SRT and scRNA-seq columns (pairwise-complete pairing)
- Every contributing method is one of the 49 methods evaluated in the benchmark (e.g. SRTsim, scDesign3, ZINB-WaVE, scDesign2)
- The output correlation_srt_scrnaseq record contains exactly the fields pearson_r, p-value, and method count

## Limitations

- The correlation is computed only over methods with complete scores in both modalities; methods with execution errors — which the paper attributes mainly to failed parameter estimations and missing or infinite values — drop out of the pairing.
- A high Pearson's r reflects overall linear agreement of accuracy scores, not that every individual method's rank is preserved across SRT and scRNA-seq datasets.
- The benchmark evaluated 49 methods on 152 reference datasets from 24 platforms; correlation strength may vary for platforms or method subsets not covered by those datasets.

## Evidence

- [abstract] Cross-modality transferability of accuracy rankings: "some methods tailored to scRNA-seq data have potential compatibility for simulating SRT data"
- [abstract] Scope of the underlying benchmark that produced the paired scores: "We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability"
- [methods] Installation of the benchmarking environment whose scores feed the correlation: "devtools::install_github("duohongrui/simmethods")"
- [readme] Pipeline that evaluates simulated data and generates the accuracy scores: "simulating single-cell RNA-seq datasets and evaluating the simulated data from general and functional perspectives."
