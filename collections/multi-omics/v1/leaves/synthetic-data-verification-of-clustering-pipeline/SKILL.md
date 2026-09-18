---
name: synthetic-data-verification-of-clustering-pipeline
description: Use when you need to verify that a co-expression clustering pipeline
  (e.g. SGCRNA's CGM + SpectralClustering) preserves known modular structure before
  applying it to real multi-omic data.
when_to_use_negative:
- The goal is biological discovery on real expression data (e.g. GSE114007 RNA counts)
  rather than verifying pipeline correctness on known ground truth.
- The clustering method explicitly relies on the scale-free topology assumption and
  manual parameter tuning, which SGCRNA was designed to avoid.
- No synthetic ground truth with known block-correlation modules exists, so cluster
  assignments cannot be compared against a reference.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: SGCRNAs
  role: Julia package supplying the CGM pre-processing function (with :LESS/:SIGMA/:FTEST
    filter modes) and the SpectralClustering function being verified
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: Julia
  role: Language used to generate the synthetic expression matrix, run the CGM and
    SpectralClustering functions, and write the verification report
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:668275adc1a6c22e126ec33dba2bcd47b265f195daa94e5ca5788c1d5b816e91
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.14285714285714285
n_claims: 7
---

# Synthetic data verification of clustering pipeline

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates the SGCRNA two-step pipeline (CGM pre-processing followed by SpectralClustering) by running it on a small synthetic expression matrix with a known block-correlation ground truth and comparing retained variables, correlation/regression matrices, and cluster assignments against that ground truth.

## When to use

Use when you need to verify that a co-expression clustering pipeline (e.g. SGCRNA's CGM + SpectralClustering) preserves known modular structure before applying it to real multi-omic data.

## When NOT to use

- The goal is biological discovery on real expression data (e.g. GSE114007 RNA counts) rather than verifying pipeline correctness on known ground truth.
- The clustering method explicitly relies on the scale-free topology assumption and manual parameter tuning, which SGCRNA was designed to avoid.
- No synthetic ground truth with known block-correlation modules exists, so cluster assignments cannot be compared against a reference.

## Inputs

- Synthetic Julia expression matrix (variables as rows, samples as columns) with known block-correlation structure
- CGM mode setting: :NONE (default), :LESS, :SIGMA, or :FTEST
- tNodeNum and pcas parameters (defaults 100 and 99; reduce for small variable counts)

## Outputs

- CorData correlation matrix and GradData regression coefficient matrix from CGM
- Frequency plots per filter mode with the removed value range highlighted in red
- clust cluster assignments, pos positions, and edge_data from SpectralClustering
- verification_report with retained/removed variable sets per mode, matrices, and cluster assignments vs ground-truth modules

## How to apply

Generate a small synthetic expression matrix in Julia (variables as rows, samples as columns) with a known block-correlation structure so the true modules are known. Run CGM with mode set to each of :LESS, :SIGMA, and :FTEST (default :NONE), confirming that for each non-:NONE mode a frequency plot is produced with the range of values to be removed highlighted in red. Check the CGM outputs — CorData (correlation matrix) and GradData (regression coefficient matrix) — against the ground truth, including exclusion of genes with near-zero expression variation or negligible regression slopes, since SGCRNA specifically addresses WGCNA's neglect of regression line slopes. Feed CorData and GradData into SpectralClustering and inspect the internal edge weighting from cor and grad, Laplacian construction, Krylov-subspace approximate eigenvector computation, and k-means cluster assignment. For a small variable count, override tNodeNum and pcas (defaults 100 and 99) to values less than roughly one-tenth of the number of variables, no less than 3. Record a verification report of the retained/removed variable sets per filter mode, the correlation and regression matrices, and the resulting clust assignments compared with the synthetic ground-truth modules.

## Related tools

- **SGCRNAs** (Julia package supplying the CGM pre-processing function (with :LESS/:SIGMA/:FTEST filter modes) and the SpectralClustering function being verified) — https://github.com/C37H41N2O6/SGCRNA
- **Julia** (Language used to generate the synthetic expression matrix, run the CGM and SpectralClustering functions, and write the verification report)

## Examples

```
CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST); clust, pos, edge_data = SpectralClustering(CorData, GradData);
```

## Evaluation signals

- Cluster assignments (clust) from SpectralClustering match the synthetic ground-truth block-correlation modules.
- For each non-:NONE mode (:LESS, :SIGMA, :FTEST), a frequency plot is generated with the range of values to be removed highlighted in red.
- CorData and GradData agree with the synthetic ground truth, including exclusion of genes with near-zero expression variation or negligible regression slopes.
- tNodeNum and pcas are set below roughly one-tenth of the variable count (minimum 3) rather than left at defaults 100 and 99.
- verification_report records the retained/removed variable sets per filter mode and both output matrices.

## Limitations

- SGCRNA itself addresses the scale-free assumption and parameter tuning, but the verification is only as good as the synthetic ground truth; small matrices require overriding tNodeNum and pcas defaults (100 and 99) designed for larger data.
- The README documentation is the primary usage reference; no changelog was found for tracking behavioral changes in SGCRNAs.jl.

## Evidence
