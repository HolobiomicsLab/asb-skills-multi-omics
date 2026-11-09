---
name: spectral-clustering-laplacian-eigendecomposition
description: Use when clustering gene-expression or multi-omic variables into co-expression
  modules without scale-free constraints, especially when regression slopes and correlation
  jointly matter and minimal parameter tuning is desired.
when_to_use_negative:
- Input is already a correlation/adjacency matrix without the raw expression data
  — CGM needs per-gene expression variation to compute regression slopes and to drop
  near-constant genes
- The analysis explicitly requires WGCNA-style scale-free topology fitting, soft-thresholding
  parameter selection, or hierarchical clustering with hub-gene scoring
- Gene count is small relative to SpectralClustering defaults — leaving tNodeNum=100
  and pcas=99 would exceed a dataset with far fewer variables
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SGCRNAs
  role: Julia package providing CGM (preprocessing/filtering) and SpectralClustering
    (Laplacian eigendecomposition and k-means clustering) used directly in this skill
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: Julia
  role: Runtime environment (Julia 1.11 or later) in which the SGCRNAs package functions
    are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:7342b0d44639df388382cac51d3e26b05ed788d904f36ba90f4c5d2846027dd0
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spectral-clustering-laplacian-eigendecomposition

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

SGCRNA's two-step co-expression workflow: the Julia CGM function pre-processes an expression matrix (computing correlation CorData and regression-coefficient GradData matrices, optionally filtering variables via mode=:LESS/:SIGMA/:FTEST), then SpectralClustering performs graph edge weighting from cor and grad, Laplacian construction, approximate eigenvector computation, and k-means module assignment — avoiding WGCNA's scale-free topology assumption.

## When to use

Use when clustering gene-expression or multi-omic variables into co-expression modules without scale-free constraints, especially when regression slopes and correlation jointly matter and minimal parameter tuning is desired.

## When NOT to use

- Input is already a correlation/adjacency matrix without the raw expression data — CGM needs per-gene expression variation to compute regression slopes and to drop near-constant genes
- The analysis explicitly requires WGCNA-style scale-free topology fitting, soft-thresholding parameter selection, or hierarchical clustering with hub-gene scoring
- Gene count is small relative to SpectralClustering defaults — leaving tNodeNum=100 and pcas=99 would exceed a dataset with far fewer variables

## Inputs

- Normalized count/expression matrix as TSV (e.g. test_data/normalizedCounts_coding.tsv) with gene symbols and sample columns
- Gene symbol vector and sample-by-sample matrix from DataFrames (Data.Symbol, Matrix(Data[:,5:end]))
- Optional phenotype/clinical table (e.g. SraRunTable.csv) for downstream module-phenotype correlation

## Outputs

- CorData: correlation matrix over retained variables
- GradData: regression coefficient (slope) matrix
- clust, pos, edge_data from SpectralClustering (cluster assignments, coordinates, edge table)
- Filter-mode diagnostic plot with removed value range highlighted in red
- Network graph PNG (e.g. Result/Fig/AllNetWork-0.5.png) and module-phenotype correlation plot (CorPhenMod.png)

## How to apply

Install SGCRNAs in Julia 1.11+ (Pkg.add("SGCRNAs")) and load the raw expression matrix (e.g. normalizedCounts_coding.tsv, genes in rows, samples in columns starting at column 5). Run CGM(Data.Symbol, Matrix(Data[:,5:end])) to obtain the CorData correlation matrix and GradData regression-coefficient matrix; pass mode=:FTEST (or :LESS/:SIGMA) to filter non-significant variables, which also saves a plot with the removed value range highlighted in red. Feed CorData and GradData into SpectralClustering, which weights graph edges from correlation and regression slope, builds the Laplacian, computes approximate eigenvectors, and returns clust, pos, and edge_data. For small variable counts, override the tNodeNum and pcas defaults (100 and 99) to values below roughly one-tenth of the variable count and no less than 3. Validate that filtered-out genes include those with near-zero expression variation or negligible regression slopes, and that resulting clusters (clust) recover known block-correlation modules on synthetic ground truth.

## Related tools

- **SGCRNAs** (Julia package providing CGM (preprocessing/filtering) and SpectralClustering (Laplacian eigendecomposition and k-means clustering) used directly in this skill) — https://github.com/C37H41N2O6/SGCRNA
- **Julia** (Runtime environment (Julia 1.11 or later) in which the SGCRNAs package functions are executed)

## Examples

```
CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST); clust, pos, edge_data = SpectralClustering(CorData, GradData);
```

## Evaluation signals

- CGM returns exactly two matrices — CorData and GradData — of matching dimensions over retained genes; genes with near-zero expression variation or negligible regression slopes are excluded
- For each non-:NONE mode (:LESS, :SIGMA, :FTEST), a plot is generated in the current directory with the removed value range highlighted in red
- SpectralClustering returns a triple (clust, pos, edge_data) with maximum(clust[d]) giving the number of modules k usable by SetNetwork
- On a synthetic matrix with known block-correlation structure, cluster assignments (clust) match the ground-truth modules
- tNodeNum and pcas are set below ~one-tenth of the variable count (minimum 3) when working with small gene sets

## Limitations

- The mode filter (:LESS/:SIGMA/:FTEST) is optional (default :NONE) and its diagnostic plot only visualizes removal ranges — retained/removed gene sets must be reconciled manually against the filtered matrices
- Defaults tNodeNum=100 and pcas=99 are unsuitable for small variable counts and must be overridden
- No changelog is available for the repository, so version-specific behavior changes cannot be tracked

## Evidence
