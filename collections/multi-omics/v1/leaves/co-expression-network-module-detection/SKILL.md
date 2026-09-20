---
name: co-expression-network-module-detection
description: Use when you need to partition genes from a normalized bulk RNA-Seq expression
  matrix into co-expression modules and compare a scale-free-constrained method (WGCNA)
  with a spectral-clustering alternative (SGCRNA).
when_to_use_negative:
- Single-cell or spatial transcriptomic data, where WGCNA variants such as hdWGCNA
  are specifically optimized instead
- Analyses where the scale-free topology assumption is known to hold and standard
  WGCNA with tuned parameters is already adequate
- Causal inference within co-expression networks, for which CWGCNA is designed rather
  than SGCRNA
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: SGCRNAs
  role: Julia package providing CGM preprocessing and SpectralClustering to build
    the correlation/regression-coefficient matrices and partition genes into modules
    without scale-free constraints
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: WGCNA
  role: R package used as the conventional baseline, run with manually tuned soft-thresholding
    power to satisfy the scale-free topology criterion
- name: Julia
  role: Runtime environment (version 1.11 or later) in which the SGCRNAs CGM and SpectralClustering
    functions are executed
  repo: https://github.com/C37H41N2O6/SGCRNA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:4d033d854b933e40993e6420c726d6d473680c2d95827732aa28db3da75fe80f
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
  - SGCRNAs
  - WGCNA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# co-expression-network-module-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detect gene co-expression modules on bulk RNA-Seq data by contrasting SGCRNA (a Julia spectral-clustering method free of scale-free topology assumptions) against conventional WGCNA run with scale-free-validating parameters.

## When to use

Use when you need to partition genes from a normalized bulk RNA-Seq expression matrix into co-expression modules and compare a scale-free-constrained method (WGCNA) with a spectral-clustering alternative (SGCRNA).

## When NOT to use

- Single-cell or spatial transcriptomic data, where WGCNA variants such as hdWGCNA are specifically optimized instead
- Analyses where the scale-free topology assumption is known to hold and standard WGCNA with tuned parameters is already adequate
- Causal inference within co-expression networks, for which CWGCNA is designed rather than SGCRNA

## Inputs

- Normalized bulk RNA-Seq count matrix (TSV, e.g. normalizedCounts_coding.tsv) with variables as rows and samples as columns
- Clinical/phenotype data table (CSV, e.g. SraRunTable.csv) for optional module-phenotype correlation

## Outputs

- Per-method module counts (module_counts.csv)
- Per-gene module assignments (module_assignments.csv)
- Co-expression network graph image (e.g. AllNetWork-0.5.png)
- Module-phenotype correlation figure (CorPhenMod.png)

## How to apply

Load the expression matrix (e.g. GSE114007's normalizedCounts_coding.tsv) in Julia with CSV.jl and DataFrames.jl, with variables as rows and samples as columns. Preprocess with SGCRNAs.CGM, passing the gene symbols and the sample columns (Data[:,5:end]) with mode=:FTEST to filter low-information variables and build the correlation and regression-coefficient matrices. Cluster with SGCRNAs.SpectralClustering using default sub-clustering parameters (tNodeNum=100, depthMax=5) and record the module count (expected: 76 on GSE114007) and per-gene labels. For the WGCNA comparison, run the R WGCNA package on the same matrix, manually tuning the soft-thresholding power and other essential parameters to satisfy the scale-free-topology criterion, yielding 14 modules on GSE114007. Tabulate per-method module counts and module-gene assignments (module_counts.csv, module_assignments.csv) so the contrast in module granularity — SGCRNA's lack of the scale-free assumption vs WGCNA's requirement for manual parameter tuning — can be validated against published values.

## Related tools

- **SGCRNAs** (Julia package providing CGM preprocessing and SpectralClustering to build the correlation/regression-coefficient matrices and partition genes into modules without scale-free constraints) — https://github.com/C37H41N2O6/SGCRNA
- **WGCNA** (R package used as the conventional baseline, run with manually tuned soft-thresholding power to satisfy the scale-free topology criterion)
- **Julia** (Runtime environment (version 1.11 or later) in which the SGCRNAs CGM and SpectralClustering functions are executed) — https://github.com/C37H41N2O6/SGCRNA

## Examples

```
using CSV, SGCRNAs; Data = CSV.read("normalizedCounts_coding.tsv", header=1, delim='\t', DataFrame); CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST); clust, pos, edge_data = SpectralClustering(CorData, GradData);
```

## Evaluation signals

- On GSE114007, SGCRNA with mode=:FTEST and default sub-clustering (tNodeNum=100, depthMax=5) yields 76 modules; a large deviation indicates misapplied preprocessing or clustering parameters
- WGCNA run with scale-free-validating parameters on the same matrix yields 14 modules
- Per-gene module labels cover all filtered genes with no unassigned entries in module_assignments.csv
- The CGM :FTEST mode generates a diagnostic plot with the removed value range highlighted in red, confirming the filter was applied
- SetNetwork/DrawNetwork output renders k = maximum(clust[d]) modules, matching the recorded module count

## Limitations

- Conventional WGCNA assumes scale-free topology, requires complex manual parameter tuning (especially the soft-thresholding power), and disregards regression coefficients, so genes with minimal expression variation but high correlation can falsely imply co-expression
- Scale-free networks are less prevalent than previously assumed, undermining WGCNA's core assumption
- SGCRNA's CGM mode defaults to :NONE; :LESS, :SIGMA, or :FTEST must be set explicitly to filter low-information variables
- No changelog is available for the SGCRNAs.jl repository

## Evidence

- [abstract] conventional algorithms manifest three principal limitations: the assumption of scale-free topology, the requirement for parameter tuning, and the neglect of regression line slopes.: "conventional algorithms manifest three principal limitations: the assumption of scale-free topology, the requirement for parameter tuning, and the neglect of regression line slopes."
- [methods] CGM: Performs preprocessing for clustering.: "CGM: Performs preprocessing for clustering."
- [methods] SpectralClustering: Carries out the clustering.: "SpectralClustering: Carries out the clustering."
- [readme] CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST);: "CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST);"
- [methods] An optional argument, `mode` (default: `:NONE`), can be set to `:LESS`, `:SIGMA`, or `:FTEST`.: "An optional argument, `mode` (default: `:NONE`), can be set to `:LESS`, `:SIGMA`, or `:FTEST`."
