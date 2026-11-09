---
name: cross-method-module-count-comparison
description: Use when comparing module granularity of co-expression methods (SGCRNA
  vs WGCNA/hdWGCNA) on the same gene or genus-level count matrix, e.g. bulk RNA-Seq,
  Visium spatial, or 16S amplicon data.
when_to_use_negative:
- Input data have already been reduced to cluster/module labels, so re-clustering
  with CGM/SpectralClustering would double-process the network
- The goal is differential expression or trait association alone, not co-expression
  module identification or method comparison
- The dataset is too small or low-information for CGM filtering to retain informative
  variables (e.g. filter modes may remove the majority of variables, as with 208 genera
  reduced to 15)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SGCRNAs
  role: Julia package providing CGM preprocessing (mode=:FTEST/:SIGMA/:LESS) and SpectralClustering
    (tNodeNum=100, depthMax=5) to produce the SGCRNA module partition
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: Julia
  role: Runtime environment (1.11 or later) with CSV.jl and DataFrames.jl for loading
    matrices and running SGCRNAs
- name: WGCNA
  role: R comparative baseline; soft-thresholding power manually tuned to satisfy
    the scale-free-topology criterion before recording the module count
- name: hdWGCNA
  role: WGCNA variant optimised for single-cell and spatial transcriptomic data, run
    on the same normalised matrix for the Visium comparison
- name: QIIME 2
  role: Denoising, feature classification, and taxonomy collapsing of 16S reads (PRJNA762466)
    to build the genus-level count matrix
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:3354910430ee89176f94ae7bc650b04fdab485610fe4d9fbfa997922ecd1436c
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
  - hdWGCNA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cross-method-module-count-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill that applies SGCRNA's Julia spectral-clustering co-expression pipeline (CGM + SpectralClustering) and a comparative co-expression method (WGCNA/hdWGCNA) to the same expression matrix, then tabulates per-method module counts and module-gene assignments to quantify how the scale-free-free approach changes module granularity.

## When to use

Use when comparing module granularity of co-expression methods (SGCRNA vs WGCNA/hdWGCNA) on the same gene or genus-level count matrix, e.g. bulk RNA-Seq, Visium spatial, or 16S amplicon data.

## When NOT to use

- Input data have already been reduced to cluster/module labels, so re-clustering with CGM/SpectralClustering would double-process the network
- The goal is differential expression or trait association alone, not co-expression module identification or method comparison
- The dataset is too small or low-information for CGM filtering to retain informative variables (e.g. filter modes may remove the majority of variables, as with 208 genera reduced to 15)

## Inputs

- Normalized gene expression matrix (TSV, e.g. normalizedCounts_coding.tsv) with variables as rows and samples as columns
- Genus-level 16S amplicon count matrix from QIIME 2 (denoised, taxonomy-collapsed)
- Seurat-normalized Visium spatial transcriptomics dataset (e.g. GSE261545)
- Clinical/phenotype metadata table (e.g. SraRunTable.csv) for module-trait correlation

## Outputs

- Per-method module counts (e.g. module_counts.csv, sgcrna_wgcna_module_counts.tsv)
- Per-gene/per-genus module assignments (e.g. module_assignments.csv)
- SGCRNA preprocessing retention count (e.g. 15 of 208 genera retained) and diagnostic filter plot
- Module-phenotype correlation heatmap from CorPhenMod (e.g. Result/Fig/CorPhenMod.png)

## How to apply

Load the expression matrix with variables as rows and samples as columns (e.g. CSV.read of normalizedCounts_coding.tsv with delim='\t' in Julia). Preprocess with SGCRNAs.CGM, selecting a filter mode (:LESS, :SIGMA, or :FTEST) — a diagnostic plot with the removed value ranges highlighted in red is saved automatically — to obtain the CorData correlation matrix and GradData regression-coefficient matrix. Cluster with SGCRNAs.SpectralClustering(CorData, GradData) using default sub-clustering parameters (tNodeNum=100, depthMax=5) and record maximum(clust[d]) as the module count. Run the comparison method (R WGCNA with manually tuned soft-thresholding power satisfying scale-free topology, or hdWGCNA for single-cell/spatial data) on the identical matrix. Tabulate module counts and per-gene/per-genus assignments (e.g. 76 vs 14 on GSE114007; 114 vs 2 on Visium GSE261545; 12 vs 8 on PRJNA762466 after SGCRNA retained 15 of 208 genera) and optionally correlate modules to phenotypes with SGCRNAs.CorPhenMod, ensuring sample/column order matches between expression and annotation tables.

## Related tools

- **SGCRNAs** (Julia package providing CGM preprocessing (mode=:FTEST/:SIGMA/:LESS) and SpectralClustering (tNodeNum=100, depthMax=5) to produce the SGCRNA module partition) — https://github.com/C37H41N2O6/SGCRNA
- **Julia** (Runtime environment (1.11 or later) with CSV.jl and DataFrames.jl for loading matrices and running SGCRNAs)
- **WGCNA** (R comparative baseline; soft-thresholding power manually tuned to satisfy the scale-free-topology criterion before recording the module count)
- **hdWGCNA** (WGCNA variant optimised for single-cell and spatial transcriptomic data, run on the same normalised matrix for the Visium comparison)
- **QIIME 2** (Denoising, feature classification, and taxonomy collapsing of 16S reads (PRJNA762466) to build the genus-level count matrix)

## Examples

```
using CSV, DataFrames, SGCRNAs; Data = CSV.read("normalizedCounts_coding.tsv", header=1, delim='\t', DataFrame); CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST); clust, pos, edge_data = SpectralClustering(CorData, GradData)
```

## Evaluation signals

- Reproduced module counts match published values: 76 (SGCRNA) vs 14 (WGCNA) on GSE114007; 114 (SGCRNA) vs 2 (hdWGCNA) on GSE261545; 12 (SGCRNA) vs 8 (WGCNA) on PRJNA762466
- SGCRNA preprocessing retention recorded (e.g. 15 of 208 genera retained after :LESS/:SIGMA/:FTEST filtering), with the saved diagnostic plot showing removed ranges in red
- Both methods run on the identical expression matrix with variables as rows and samples as columns; column order matches the phenotype table before CorPhenMod
- WGCNA soft-thresholding power documented as satisfying the scale-free-topology criterion
- Output files (module_counts.csv, module_assignments.csv) contain module counts consistent with maximum(clust[d]) from SpectralClustering output

## Limitations

- WGCNA requires complex manual parameter tuning (two essential parameters), with the second parameter in particular requiring considerable effort to optimise
- WGCNA assumes scale-free topology, which recent studies suggest is less prevalent than previously assumed
- WGCNA disregards regression coefficients, so a gene with minimal expression variation but high correlation coefficient may falsely imply co-expression
- CGM filter modes (:LESS, :SIGMA, :FTEST) remove variables — a diagnostic plot with removed ranges highlighted in red is generated and should be inspected

## Evidence
