---
name: co-expression-matrix-low-variance-filtering
description: Use when an expression matrix (variables × samples, e.g. normalizedCounts_coding.tsv)
  feeds a co-expression pipeline and genes with minimal expression variation or negligible
  regression slopes must be removed before clustering.
when_to_use_negative:
- The input has already been quality-filtered and contains no genes with near-zero
  expression variation — the filter step adds nothing.
- The goal is to reproduce classic WGCNA behavior with scale-free topology and soft
  thresholding rather than SGCRNA's correlation-plus-regression-slope preprocessing.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: SGCRNAs
  role: Provides the CGM Julia function that performs the pre-processing/filtering
    and returns CorData and GradData
  repo: https://github.com/C37H41N2O6/SGCRNAs
- name: Julia
  role: Runtime environment (Julia 1.11 or later) in which the SGCRNAs package is
    installed and CGM filtering is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:88937f0548e068cfaa35dbde15eed4f6def4623dc38870a3313a46c84d829e60
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/C37H41N2O6/SGCRNAs.jl
  repo_url: https://github.com/C37H41N2O6/SGCRNAs
  tools:
  - Julia
  - SGCRNAs
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# co-expression-matrix-low-variance-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A pre-processing step from SGCRNA's CGM function that excludes non-significant variables — genes with near-zero expression variation or negligible regression slopes — from co-expression analysis before spectral clustering, countering WGCNA's disregard of regression coefficients.

## When to use

Use when an expression matrix (variables × samples, e.g. normalizedCounts_coding.tsv) feeds a co-expression pipeline and genes with minimal expression variation or negligible regression slopes must be removed before clustering.

## When NOT to use

- The input has already been quality-filtered and contains no genes with near-zero expression variation — the filter step adds nothing.
- The goal is to reproduce classic WGCNA behavior with scale-free topology and soft thresholding rather than SGCRNA's correlation-plus-regression-slope preprocessing.

## Inputs

- Expression matrix as TSV/DataFrame (e.g. normalizedCounts_coding.tsv with header row, '#' comments, tab delimiter), variables as rows, samples as columns
- Gene symbol column (e.g. Data.Symbol) passed alongside the numeric sample columns
- Optional filter mode setting: :LESS, :SIGMA, or :FTEST (default :NONE)

## Outputs

- CorData: correlation matrix over retained variables
- GradData: regression coefficient matrix over retained variables
- Frequency plot saved in the current directory with the removed value range highlighted in red (when mode is non-:NONE)
- Retained vs. removed variable sets, feeding downstream SpectralClustering

## How to apply

Run the CGM function from the SGCRNAs Julia package on the expression matrix, passing gene symbols and the sample columns (e.g. columns 5:end of the TSV). Optionally set the `mode` argument (default :NONE) to :LESS, :SIGMA, or :FTEST; any of these modes triggers a frequency plot saved in the current directory with the range of values to be removed highlighted in red, providing a visual check on the cut. The filtering rationale is that a gene with minimal expression variation can still show a high correlation coefficient, which falsely implies co-expression. Verify the retained/removed sets — genes with near-zero expression variation or negligible regression slopes should be excluded — then pass the resulting CorData (correlation matrix) and GradData (regression coefficient matrix) downstream to SpectralClustering.

## Related tools

- **SGCRNAs** (Provides the CGM Julia function that performs the pre-processing/filtering and returns CorData and GradData) — https://github.com/C37H41N2O6/SGCRNAs
- **Julia** (Runtime environment (Julia 1.11 or later) in which the SGCRNAs package is installed and CGM filtering is executed)

## Examples

```
using SGCRNAs; CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST)
```

## Evaluation signals

- For every non-:NONE filter mode (:LESS, :SIGMA, :FTEST), a frequency plot is produced in the current directory with the removed value range highlighted in red.
- Genes with near-zero expression variation or negligible regression slopes appear in the removed set, not in the retained CorData/GradData columns.
- Row/column dimensionality of CorData and GradData matches the count of retained variables, and both matrices are symmetric/aligned over the same gene symbols.
- Downstream SpectralClustering accepts the filtered matrices without error, and cluster assignments (clust) can be compared against a known block-correlation ground truth on synthetic data.
- When tNodeNum/pcas are reduced for small variable counts (values no less than 3, below ~one-tenth of variables instead of defaults 100 and 99), filtering and clustering still complete.

## Limitations

- With the default mode :NONE, no filtering plot is generated and non-significant variables are not excluded — the mode must be explicitly set to :LESS, :SIGMA, or :FTEST.
- Filtering criteria are tied to SGCRNA's CGM implementation; the removed value range must be inspected visually via the red-highlighted frequency plot rather than a fixed numeric threshold stated in the paper.

## Evidence

- [methods] Optional mode argument: "An optional argument, `mode` (default: `:NONE`), can be set to `:LESS`, `:SIGMA`, or `:FTEST`."
- [methods] Removal-range plot: "If any of these modes are specified, a plot will be generated and saved in the current directory, with the range of values to be removed highlighted in red."
- [intro] Rationale for filtering: "Even when a gene exhibits minimal variation in expression, a high correlation coefficient may still falsely imply co-expression."
- [methods] CGM pre-processing role: "Preprocessing of the data is performed using the CGM function."
- [readme] README invocation of CGM filtering: "CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST);"
