---
name: wgcna-comparative-module-inference
description: Use when a co-expression/co-abundance matrix (e.g. genus-level counts
  from QIIME 2) must be partitioned into modules by both SGCRNA and WGCNA to compare
  module granularity and preprocessing retention.
when_to_use_negative:
- Input is already a pre-filtered feature table with no low-information variables,
  so the CGM `mode` filter (:LESS/:SIGMA/:FTEST) has nothing to remove
- The goal is differential expression or taxonomy assignment rather than module detection
  from correlated expression patterns
- Data must be analysed under a scale-free topology assumption or with WGCNA's soft-thresholding
  parameter tuning specifically being the object of study
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3174
tools:
- name: SGCRNAs
  role: Julia package providing CGM preprocessing (with :LESS/:SIGMA/:FTEST filter
    modes) and SpectralClustering to infer co-expression modules without scale-free
    constraints
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: WGCNA
  role: Comparative baseline method run on the same matrix to yield the reference
    module count
- name: QIIME 2
  role: Denoising, feature classification and taxonomy collapsing of 16S amplicon
    reads to produce the genus-level count matrix
- name: Julia
  role: Runtime environment (1.11 or later) for executing SGCRNAs.jl functions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:abdcc6ecbfa6e7d69c56c4e00575a357314d500dfb3fea9842f53aec49cac424
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# wgcna-comparative-module-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark SGCRNA (spectral clustering-guided co-expression network analysis) against WGCNA on the same feature matrix to compare module counts and preprocessing retention, as demonstrated on PRJNA762466 16S genus-level data (15 genera retained, 12 SGCRNA modules vs 8 WGCNA modules).

## When to use

Use when a co-expression/co-abundance matrix (e.g. genus-level counts from QIIME 2) must be partitioned into modules by both SGCRNA and WGCNA to compare module granularity and preprocessing retention.

## When NOT to use

- Input is already a pre-filtered feature table with no low-information variables, so the CGM `mode` filter (:LESS/:SIGMA/:FTEST) has nothing to remove
- The goal is differential expression or taxonomy assignment rather than module detection from correlated expression patterns
- Data must be analysed under a scale-free topology assumption or with WGCNA's soft-thresholding parameter tuning specifically being the object of study

## Inputs

- Genus-level count matrix (variables as rows, samples as columns) derived from QIIME 2 processing of 16S amplicon reads (e.g. PRJNA762466 from SRA)
- Tab-delimited normalized expression/feature count table (e.g. normalizedCounts_coding.tsv) with a symbol/identifier column
- Optional sample/phenotype metadata table (e.g. SraRunTable.csv) for downstream module-trait correlation via CorPhenMod

## Outputs

- Filtered variable set from CGM (e.g. 15 of 208 genera retained) plus a diagnostic plot with removed ranges highlighted in red
- SGCRNA clustering assignments stored in `clust` (module count, e.g. 12 modules)
- WGCNA module assignments on the same matrix (module count, e.g. 8 modules)
- Compiled comparison table (e.g. sgcrna_wgcna_module_counts.tsv)

## How to apply

Build a variables-as-rows, samples-as-columns count matrix (e.g. genus-level table from QIIME 2 denoising, feature classification and taxonomy collapsing). Run SGCRNA's CGM preprocessing function on the matrix, optionally setting its `mode` argument to :LESS, :SIGMA, or :FTEST (default :NONE) to filter low-information variables — CGM also writes a diagnostic plot with the removed value ranges highlighted in red. Feed CGM's correlation matrix and regression coefficient matrix outputs into SGCRNA's SpectralClustering to obtain clustering results in `clust`; spectral clustering avoids WGCNA's scale-free-topology assumption and manual parameter tuning, and exploits regression line slopes that WGCNA disregards. Run WGCNA on the identical matrix as the comparative baseline, then tabulate the retained-variable count and both methods' module counts (e.g. sgcrna_wgcna_module_counts.tsv) and check them against expected values.

## Related tools

- **SGCRNAs** (Julia package providing CGM preprocessing (with :LESS/:SIGMA/:FTEST filter modes) and SpectralClustering to infer co-expression modules without scale-free constraints) — https://github.com/C37H41N2O6/SGCRNA
- **WGCNA** (Comparative baseline method run on the same matrix to yield the reference module count)
- **QIIME 2** (Denoising, feature classification and taxonomy collapsing of 16S amplicon reads to produce the genus-level count matrix)
- **Julia** (Runtime environment (1.11 or later) for executing SGCRNAs.jl functions)

## Examples

```
using SGCRNAs; CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST); clust, pos, edge_data = SpectralClustering(CorData, GradData);
```

## Evaluation signals

- Retention count matches expectation: e.g. 15 of 208 genera retained after SGCRNA pre-processing on PRJNA762466
- SGCRNA module count equals the reported value (12 modules) for the PRJNA762466 genus matrix
- WGCNA baseline module count equals the reported value (8 modules) on the identical matrix
- CGM with a filter mode writes a diagnostic plot in the current directory with the value range removed highlighted in red
- SpectralClustering returns clust/pos/edge_data tuple and maximum(clust[d]) equals the number of modules used in SetNetwork

## Limitations

- Conventional WGCNA assumes scale-free topology, which recent studies suggest is less prevalent than previously assumed
- WGCNA requires complex manual parameter tuning, with the second parameter in particular requiring considerable effort to optimise
- WGCNA disregards regression coefficients, so a gene with minimal expression variation but high correlation coefficient may falsely imply co-expression
- No changelog found for the SGCRNAs.jl repository

## Evidence
