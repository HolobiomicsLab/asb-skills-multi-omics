---
name: spectral-clustering-network-analysis
description: Use when clustering genes (or other multi-omic variables) into co-expression
  modules from a normalized expression matrix and the scale-free topology assumption
  of WGCNA is inappropriate or manual parameter tuning should be avoided.
when_to_use_negative:
- When the scale-free topology assumption is valid for the network and the standard
  WGCNA workflow with tuned soft-thresholding power is desired
- When the input is raw (not normalized) count data rather than a normalized expression
  matrix
- When only one global cluster is needed without sub-clustering structure — SGCRNA's
  hierarchical spectral sub-clustering (tNodeNum=100, depthMax=5) is designed for
  recursive module discovery
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: SGCRNAs
  role: Julia package providing CGM (preprocessing), SpectralClustering (module clustering),
    SetNetwork, DrawNetwork, and CorPhenMod functions
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: Julia
  role: Runtime environment (version 1.11 or later) in which SGCRNAs is installed
    via Pkg and the pipeline is executed
- name: WGCNA
  role: Baseline co-expression method compared against SGCRNA module counts (14 vs
    76 modules on GSE114007)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:54b26591515f6125ca2cc9f38e67966b502460f56fd4589fb71238aee3cb7a1b
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# spectral-clustering-network-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply SGCRNA, a Julia implementation of spectral clustering-guided co-expression network analysis, to identify gene modules without WGCNA's scale-free topology assumption, manual soft-threshold tuning, or disregard of regression line slopes.

## When to use

Use when clustering genes (or other multi-omic variables) into co-expression modules from a normalized expression matrix and the scale-free topology assumption of WGCNA is inappropriate or manual parameter tuning should be avoided.

## When NOT to use

- When the scale-free topology assumption is valid for the network and the standard WGCNA workflow with tuned soft-thresholding power is desired
- When the input is raw (not normalized) count data rather than a normalized expression matrix
- When only one global cluster is needed without sub-clustering structure — SGCRNA's hierarchical spectral sub-clustering (tNodeNum=100, depthMax=5) is designed for recursive module discovery

## Inputs

- Normalized gene expression matrix as TSV (e.g. normalizedCounts_coding.tsv) with variables as rows and samples as columns
- Clinical/phenotype data table (e.g. SraRunTable.csv) for module-trait correlation
- Gene symbol column (e.g. Data.Symbol) accompanying the sample matrix

## Outputs

- Per-gene module cluster labels from SpectralClustering (clust vector)
- Correlation and regression-coefficient (gradient) matrices from CGM (CorData, GradData)
- Network graph visualization (e.g. AllNetWork-0.5.png)
- Module-phenotype correlation figure (e.g. CorPhenMod.png)

## How to apply

Load the expression matrix (e.g. GSE114007's normalizedCounts_coding.tsv) in Julia with CSV.jl/DataFrames.jl, variables as rows and samples as columns. Preprocess with the SGCRNAs CGM function, passing gene symbols and the sample matrix with an optional filter mode such as :FTEST (alternatives :LESS or :SIGMA; a diagnostic plot with removed ranges highlighted in red is generated). Cluster with SpectralClustering(CorData, GradData) using default sub-clustering parameters (tNodeNum=100, depthMax=5), which returns per-gene module labels, positions, and edge data. Optionally visualize with SetNetwork/DrawNetwork (edge_threshold e.g. 0.5) and correlate modules with phenotypes via CorPhenMod using SraRunTable.csv clinical data. Rationale: spectral clustering does not depend on the scale-free assumption, whereas WGCNA on the same GSE114007 data with scale-free-validating parameters yielded far fewer modules (14 vs SGCRNA's 76).

## Related tools

- **SGCRNAs** (Julia package providing CGM (preprocessing), SpectralClustering (module clustering), SetNetwork, DrawNetwork, and CorPhenMod functions) — https://github.com/C37H41N2O6/SGCRNA
- **Julia** (Runtime environment (version 1.11 or later) in which SGCRNAs is installed via Pkg and the pipeline is executed)
- **WGCNA** (Baseline co-expression method compared against SGCRNA module counts (14 vs 76 modules on GSE114007))

## Examples

```
using CSV, DataFrames, SGCRNAs; Data = CSV.read("normalizedCounts_coding.tsv", header=1, delim='\t', DataFrame); CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST); clust, pos, edge_data = SpectralClustering(CorData, GradData);
```

## Evaluation signals

- On GSE114007 bulk OA RNA-Seq, SGCRNA should partition genes into ~76 modules (vs 14 for scale-free-tuned WGCNA)
- CGM with mode=:FTEST must emit a diagnostic plot in the current directory with removed value ranges highlighted in red
- SpectralClustering returns three objects (clust, pos, edge_data); maximum(clust[d]) should equal the number of modules k used in SetNetwork
- Per-gene module assignments should be tabulated and match the published module-gene assignments on the same input matrix
- The Julia environment must be 1.11 or later and SGCRNAs installed via Pkg.add("SGCRNAs")

## Limitations

- Conventional WGCNA-style approaches assume scale-free topology, which recent studies suggest is less prevalent than previously assumed; SGCRNA removes this assumption but results (module counts) differ substantially from WGCNA on the same data
- Filter mode choices (:NONE default, :LESS, :SIGMA, :FTEST) alter which low-information variables are removed and thus affect module structure
- Repository has no changelog ('_No changelog found._'), so version drift in sub-clustering defaults may need checking

## Evidence
