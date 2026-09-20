---
name: coexpression-network-preprocessing-filtering
description: Use when a raw genus-level or gene-level count matrix (variables as rows,
  samples as columns) must be filtered of low-information variables before co-expression
  module detection with SGCRNA or comparison against WGCNA.
when_to_use_negative:
- Input is already a pre-filtered correlation/adjacency matrix rather than a raw count
  matrix — CGM expects the original variable-by-sample matrix
- The analysis plan requires the scale-free topology assumption and manual soft-threshold
  parameter tuning of conventional WGCNA rather than SGCRNA's spectral clustering
- The dataset is single-cell or spatial transcriptomics where hdWGCNA-style variants
  are the intended pipeline
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3174
tools:
- name: SGCRNAs
  role: Julia package whose CGM function performs the preprocessing/filtering and
    returns CorData and GradData for clustering
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: Julia
  role: Runtime language; requires Julia 1.11 or later and Pkg.add("SGCRNAs")
- name: QIIME 2
  role: Produces the genus-level 16S count matrix (denoising, feature classification,
    taxonomy collapsing) that is the input to CGM
- name: WGCNA
  role: Comparative baseline run on the same filtered matrix to contrast module counts
    (8 modules vs SGCRNA's 12)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:af89b1b33ba600df6e5b794fe596a536eedca99449477751d24653ef24f99b0a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/C37H41N2O6/SGCRNAs.jl
  repo_url: https://github.com/C37H41N2O6/SGCRNA
  tools:
  - Julia
  - QIIME 2
  - SGCRNAs
  - WGCNA
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# coexpression-network-preprocessing-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocessing/filtering of count or expression matrices with SGCRNA's CGM function (Julia) prior to spectral-clustering co-expression network analysis, retaining only informative variables (e.g. 15 of 208 genera on a 16S genus-level matrix).

## When to use

Use when a raw genus-level or gene-level count matrix (variables as rows, samples as columns) must be filtered of low-information variables before co-expression module detection with SGCRNA or comparison against WGCNA.

## When NOT to use

- Input is already a pre-filtered correlation/adjacency matrix rather than a raw count matrix — CGM expects the original variable-by-sample matrix
- The analysis plan requires the scale-free topology assumption and manual soft-threshold parameter tuning of conventional WGCNA rather than SGCRNA's spectral clustering
- The dataset is single-cell or spatial transcriptomics where hdWGCNA-style variants are the intended pipeline

## Inputs

- Variable-by-sample count matrix as TSV (e.g. genus-level QIIME 2 output with variables as rows and samples as columns, first column holding variable names such as Data.Symbol)
- Normalized expression matrix (e.g. assets/normalizedCounts_coding.tsv)
- Choice of filter mode: :LESS, :SIGMA, or :FTEST

## Outputs

- CorData: filtered correlation matrix suitable for SpectralClustering
- GradData: regression coefficient matrix suitable for SpectralClustering
- Diagnostic PNG plot saved in the current directory showing the value ranges removed (highlighted in red)
- Filtered list of retained variables (e.g. 15 of 208 genera) for reporting to sgcrna_wgcna_module_counts.tsv

## How to apply

Load the variable-by-sample count matrix (e.g. a TSV produced by QIIME 2 taxonomy collapsing, or normalizedCounts_coding.tsv) into a Julia DataFrame. Call SGCRNAs.CGM with the variable names vector and the sample matrix, setting the optional `mode` argument to one of :LESS, :SIGMA, or :FTEST (default :NONE) to remove uninformative rows; the chosen mode also saves a diagnostic plot in the current directory with the range of removed values highlighted in red. CGM returns the correlation matrix (CorData) and regression coefficient matrix (GradData) that feed directly into SpectralClustering. Rationale: preprocessing discards variables whose expression variation or regression behaviour would otherwise falsely imply co-expression, and in the PRJNA762466 16S benchmark it retained 15 of 208 genera. Inspect the saved diagnostic plot to confirm the filtered ranges are sensible before clustering.

## Related tools

- **SGCRNAs** (Julia package whose CGM function performs the preprocessing/filtering and returns CorData and GradData for clustering) — https://github.com/C37H41N2O6/SGCRNA
- **Julia** (Runtime language; requires Julia 1.11 or later and Pkg.add("SGCRNAs"))
- **QIIME 2** (Produces the genus-level 16S count matrix (denoising, feature classification, taxonomy collapsing) that is the input to CGM)
- **WGCNA** (Comparative baseline run on the same filtered matrix to contrast module counts (8 modules vs SGCRNA's 12))

## Examples

```
CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST);
```

## Evaluation signals

- CGM returns exactly two outputs (CorData, GradData) and accepts variables as rows, samples as columns — verify matrix orientation matches Data.Symbol + Matrix(Data[:,5:end]) pattern
- When a mode is specified, a diagnostic plot is saved in the current directory with removed value ranges highlighted in red — check this file exists
- Retention count matches the benchmark: 15 of 208 genera retained after SGCRNA pre-processing on PRJNA762466
- Downstream SpectralClustering on the CGM outputs yields 12 modules, versus 8 from WGCNA on the same matrix
- Package installs cleanly via Pkg.add("SGCRNAs") under Julia 1.11 or later

## Limitations

- The filter `mode` argument defaults to :NONE; if no mode is set, no filtering diagnostic plot is generated
- Filter mode choice (:LESS, :SIGMA, :FTEST) is a decision point with no automatic selection in the documentation
- A variable with minimal expression variation but a high correlation coefficient may falsely imply co-expression if regression slopes are neglected — the rationale for CGM's regression-aware filtering

## Evidence

- [methods] An optional argument, `mode` (default: `:NONE`), can be set to `:LESS`, `:SIGMA`, or `:FTEST`.: "An optional argument, `mode` (default: `:NONE`), can be set to `:LESS`, `:SIGMA`, or `:FTEST`."
- [methods] If any of these modes are specified, a plot will be generated and saved in the current directory, with the range of values to be removed highlighted in red.: "If any of these modes are specified, a plot will be generated and saved in the current directory, with the range of values to be removed highlighted in red."
- [readme] CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST);: "CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST);"
- [intro] Even when a gene exhibits minimal variation in expression, a high correlation coefficient may still falsely imply co-expression.: "Even when a gene exhibits minimal variation in expression, a high correlation coefficient may still falsely imply co-expression."
- [methods] Download Julia 1.11 or later, preferably the current stable release.: "Download Julia 1.11 or later, preferably the current stable release."
