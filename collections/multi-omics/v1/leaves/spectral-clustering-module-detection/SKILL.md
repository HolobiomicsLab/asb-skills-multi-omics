---
name: spectral-clustering-module-detection
description: Use when you need to partition gene or genus expression data into co-expression
  modules without assuming scale-free topology, especially when WGCNA yields too few
  modules or requires heavy parameter tuning.
when_to_use_negative:
- When a scale-free topology assumption is acceptable and WGCNA's parameterised approach
  already yields adequate module resolution
- When the input is raw reads or unnormalised counts — normalisation (e.g. Seurat
  spatial workflow, QIIME 2 denoising and taxonomy collapsing) must precede CGM
- When features have not been pre-filtered and low-variation genes would inflate the
  correlation matrix
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SGCRNAs
  role: Julia package providing CGM preprocessing (correlation and regression-coefficient
    matrices) and SpectralClustering for module detection without scale-free constraints
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: Julia
  role: Runtime environment (version 1.11 or later) in which SGCRNAs functions are
    executed
- name: WGCNA
  role: Conventional co-expression baseline against which SGCRNA module counts are
    compared (e.g. 8 vs 12 modules on PRJNA762466)
- name: hdWGCNA
  role: Single-cell/spatial-transcriptomic WGCNA variant used as comparative baseline
    on Visium kidney data (2 modules vs SGCRNA's 114)
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:4ac8d5aa49f32afcdd7ac34c6f985e5ae363bf7b6f811533cb7048fba863729c
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# spectral-clustering-module-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detect co-expression modules using SGCRNA's spectral clustering workflow in Julia, which replaces WGCNA's scale-free-topology assumption and manual parameter tuning with correlation-matrix plus regression-coefficient (gradient) inputs.

## When to use

Use when you need to partition gene or genus expression data into co-expression modules without assuming scale-free topology, especially when WGCNA yields too few modules or requires heavy parameter tuning.

## When NOT to use

- When a scale-free topology assumption is acceptable and WGCNA's parameterised approach already yields adequate module resolution
- When the input is raw reads or unnormalised counts — normalisation (e.g. Seurat spatial workflow, QIIME 2 denoising and taxonomy collapsing) must precede CGM
- When features have not been pre-filtered and low-variation genes would inflate the correlation matrix

## Inputs

- Normalised gene/genus expression matrix (e.g. TSV with features as rows, samples as columns, such as normalizedCounts_coding.tsv)
- Feature symbol/identifier vector (e.g. Data.Symbol column)
- Optional phenotype or spatial annotation table (e.g. SraRunTable.csv) for module correlation

## Outputs

- CorData (correlation matrix) and GradData (regression-coefficient matrix) from CGM
- clust clustering assignment of features into modules
- Diagnostic filter plot with removed value ranges highlighted in red
- Network graph PNG (e.g. AllNetWork-0.5.png) and module-phenotype correlation figure (CorPhenMod.png)

## How to apply

Normalise the expression matrix first (e.g. Seurat spatial workflow for Visium data, or QIIME 2 taxonomy collapsing for 16S genus-level counts with variables as rows and samples as columns). Run SGCRNAs.CGM on the feature symbols and count matrix with an optional filter mode (:LESS, :SIGMA, or :FTEST) — CGM returns the correlation matrix (CorData) and regression-coefficient matrix (GradData), and also generates a diagnostic plot highlighting in red the range of values removed. Then run SGCRNAs.SpectralClustering(CorData, GradData) with defaults (tNodeNum=100, depthMax=5) to obtain the clustering stored in `clust`; SGCRNA partitioned 15 retained genera into 12 modules on PRJNA762466 and 114 modules on the Visium kidney dataset GSE261545, versus only 8 and 2 modules for WGCNA/hdWGCNA. Optionally visualise with SetNetwork/DrawNetwork and correlate modules with phenotypes or spatial annotations via CorPhenMod, ensuring column order of the expression matrix matches the annotation table.

## Related tools

- **SGCRNAs** (Julia package providing CGM preprocessing (correlation and regression-coefficient matrices) and SpectralClustering for module detection without scale-free constraints) — https://github.com/C37H41N2O6/SGCRNA
- **Julia** (Runtime environment (version 1.11 or later) in which SGCRNAs functions are executed)
- **WGCNA** (Conventional co-expression baseline against which SGCRNA module counts are compared (e.g. 8 vs 12 modules on PRJNA762466))
- **hdWGCNA** (Single-cell/spatial-transcriptomic WGCNA variant used as comparative baseline on Visium kidney data (2 modules vs SGCRNA's 114))

## Examples

```
using SGCRNAs; CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST); clust, pos, edge_data = SpectralClustering(CorData, GradData);
```

## Evaluation signals

- Module count comparison against baselines: expected ~12 modules (SGCRNA) vs 8 (WGCNA) on the 15 retained genera of PRJNA762466, and 114 modules (SGCRNA) vs 2 (hdWGCNA grey/turquoise) on GSE261545
- CGM with mode=:FTEST saves a plot in the current directory with the range of removed values highlighted in red — confirm it is generated
- On GSE261545, ~71% of SGCRNA modules should be non-cell-type-specific, with the remainder specific to tumour, healthy proximal tubules, glomeruli, LH-CD, and artery regions
- hdWGCNA baseline reproduces grey=tumour and turquoise=proximal tubule assignments; SGCRNA spatial enrichments are tabulated in module_comparison_sgcrna_hdwgcna.csv
- SpectralClustering output `clust` assigns every retained feature to exactly one module (k = maximum(clust[d]) defines module count)

## Limitations

- Conventional WGCNA assumptions (scale-free topology, manual parameter tuning, neglect of regression slopes) motivated SGCRNA, but SGCRNA's own parameter choices (e.g. filter mode :LESS/:SIGMA/:FTEST) still affect which features are retained — 15 of 208 genera survived pre-processing on PRJNA762466
- For module-phenotype correlation (CorPhenMod), sample/column order of the expression matrix must match the annotation table (e.g. sort Phen and innerjoin on Symbol with order=:right), otherwise correlations are misaligned
- In hdWGCNA comparison, only two modules (grey, turquoise) were recovered, illustrating that baseline methods may under-resolve modules rather than SGCRNA being directly comparable in granularity

## Evidence
