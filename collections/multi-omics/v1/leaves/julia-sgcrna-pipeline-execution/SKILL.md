---
name: julia-sgcrna-pipeline-execution
description: Use when analyzing a gene/multi-omic expression matrix (e.g. RNA normalized
  counts in TSV) to identify co-expression modules without assuming scale-free topology
  or hand-tuning WGCNA's soft-thresholding parameters.
when_to_use_negative:
- The analysis requires WGCNA's scale-free topology framework or is benchmarking WGCNA-specific
  outputs such as soft-thresholding power selection
- Runtime is the primary constraint — SGCRNA's spectral-clustering eigenvector computation
  takes substantially longer than WGCNA on the same dataset (GSE114007)
- The input is not an expression/count matrix with identifiable gene symbols and sample
  columns (e.g. already a clustered module assignment or a network edge list)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: SGCRNAs.jl
  role: Julia package providing CGM, SpectralClustering, SetNetwork, DrawNetwork,
    and CorPhenMod functions for the spectral-clustering co-expression pipeline
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: Julia 1.11
  role: Runtime environment; the article instructs downloading Julia 1.11 or later,
    preferably the current stable release
- name: WGCNA
  role: Conventional hierarchical-clustering co-expression method used as the comparison
    baseline; faster but assumes scale-free topology and requires manual parameter
    tuning
- name: CSV.jl / DataFrames.jl
  role: Loading the TSV expression matrix and CSV phenotype table into DataFrames
    within the pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:977b5f323400f011de2ac5ef03e2773562d682c26e2af10471b2e65af17e106f
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
  - CSV.jl / DataFrames.jl
  - Julia
  - Julia 1.11
  - SGCRNA
  - SGCRNAs.jl
  - WGCNA
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.36363636363636365
n_claims: 11
---

# julia-sgcrna-pipeline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs the SGCRNA co-expression analysis pipeline in Julia (CGM preprocessing, SpectralClustering, SetNetwork, DrawNetwork) on a TSV gene-expression matrix, offering a spectral-clustering alternative to WGCNA that avoids scale-free topology assumptions and manual parameter tuning.

## When to use

Use when analyzing a gene/multi-omic expression matrix (e.g. RNA normalized counts in TSV) to identify co-expression modules without assuming scale-free topology or hand-tuning WGCNA's soft-thresholding parameters.

## When NOT to use

- The analysis requires WGCNA's scale-free topology framework or is benchmarking WGCNA-specific outputs such as soft-thresholding power selection
- Runtime is the primary constraint — SGCRNA's spectral-clustering eigenvector computation takes substantially longer than WGCNA on the same dataset (GSE114007)
- The input is not an expression/count matrix with identifiable gene symbols and sample columns (e.g. already a clustered module assignment or a network edge list)

## Inputs

- Gene expression matrix in TSV format (normalizedCounts_coding.tsv), variables as rows and samples as columns starting at column 5
- Clinical/phenotype table (SraRunTable.csv, comma-delimited) for module-phenomenon correlation
- Julia 1.11 environment with SGCRNAs, CSV, and DataFrames packages installed

## Outputs

- Correlation and gradient data from CGM (CorData, GradData)
- Cluster assignments, node positions, and edge data from SpectralClustering
- Network graph image (e.g. Result/Fig/AllNetWork-0.5.png) from DrawNetwork
- Module-phenomenon correlation figure (e.g. Result/Fig/CorPhenMod.png)
- Filtered-value diagnostic plot generated when a CGM mode (:FTEST/:LESS/:SIGMA) is set

## How to apply

Load the expression matrix (e.g. normalizedCounts_coding.tsv) with CSV.jl/DataFrames.jl in Julia 1.11 so genes are rows and samples are columns. Preprocess with CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST), which generates correlation and gradient data plus a diagnostic plot highlighting removed values in red; :FTEST can be swapped for :LESS or :SIGMA as the filtering mode. Cluster the CGM output with SpectralClustering (default tNodeNum=100, depthMax=5) to obtain clusters, node positions, and edge data, then build and draw the network via SetNetwork and DrawNetwork (e.g. edge_threshold=0.5, edge_mode=:ALL). Optionally quantify module-trait association with CorPhenMod using clinical data such as SraRunTable.csv, joining on gene Symbol. Note that spectral clustering's eigenvector computation makes SGCRNA slower than WGCNA; if runtime is critical, WGCNA is faster on the same input.

## Related tools

- **SGCRNAs.jl** (Julia package providing CGM, SpectralClustering, SetNetwork, DrawNetwork, and CorPhenMod functions for the spectral-clustering co-expression pipeline) — https://github.com/C37H41N2O6/SGCRNA
- **Julia 1.11** (Runtime environment; the article instructs downloading Julia 1.11 or later, preferably the current stable release)
- **WGCNA** (Conventional hierarchical-clustering co-expression method used as the comparison baseline; faster but assumes scale-free topology and requires manual parameter tuning)
- **CSV.jl / DataFrames.jl** (Loading the TSV expression matrix and CSV phenotype table into DataFrames within the pipeline)

## Examples

```
CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST); clust, pos, edge_data = SpectralClustering(CorData, GradData);
```

## Evaluation signals

- CGM with mode=:FTEST writes a plot in the current directory with the range of removed values highlighted in red — confirm this file appears
- SpectralClustering returns three objects (clust, pos, edge_data); maximum(clust[d]) must equal the number of modules k passed to SetNetwork
- DrawNetwork produces Result/Fig/AllNetWork-0.5.png with edge_threshold=0.5 and edge_mode=:ALL
- CorPhenMod output figure is generated after innerjoin of the expression data with phenotype columns [2,3,5] of SraRunTable.csv on :Symbol
- When benchmarking against WGCNA on GSE114007, the mean SGCRNA runtime over 10 runs should be substantially larger than the mean WGCNA runtime

## Limitations

- SGCRNA requires substantially more execution time than WGCNA due to spectral clustering's eigenvector computation
- CGM filtering modes (:LESS, :SIGMA, :FTEST) alter which values are removed and each generates a plot in the current directory; the default mode is :NONE (no filtering)
- The README example assumes the expression matrix has gene symbols in the 'Symbol' column and expression values from column 5 onward, with depth d=1 clustering level

## Evidence
