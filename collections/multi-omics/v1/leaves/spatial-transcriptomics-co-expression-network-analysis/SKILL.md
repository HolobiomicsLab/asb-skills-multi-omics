---
name: spatial-transcriptomics-co-expression-network-analysis
description: Use when analysing normalised spatial transcriptomics matrices (e.g.
  Visium GSE261545) for cell type-specific co-expression modules, or when WGCNA/hdWGCNA
  over-merges modules (grey/turquoise) and finer spatial module patterns are needed.
when_to_use_negative:
- The analysis goal requires the scale-free topology assumption or hub-gene ranking
  within classical WGCNA's framework — SGCRNA explicitly discards scale-free constraints
- Data is bulk RNA-seq with known trait association needs handled fine by standard
  WGCNA/PyWGCNA and no spatial or single-cell structure to exploit
- Only a coarse two-module (grey/turquoise) partition is desired; SGCRNA's higher
  module resolution would over-split
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3512
tools:
- name: SGCRNA
  role: 'Core Julia package: CGM preprocessing (mode=:FTEST), SpectralClustering module
    discovery, SetNetwork/DrawNetwork visualisation, CorPhenMod module-phenotype correlation'
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: Julia 1.11
  role: Runtime environment; install SGCRNAs via Pkg.add("SGCRNAs")
- name: hdWGCNA
  role: Benchmark WGCNA variant optimised for single-cell and spatial transcriptomic
    data, run on the same normalised matrix for module-count comparison
- name: WGCNA
  role: Baseline co-expression method whose scale-free assumption and parameter tuning
    SGCRNA is designed to avoid
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:f0a16a668417600b3bccbd088e2ea62325938980285eaeb3861ae96e8bc6cc17
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
  - Julia 1.11
  - SGCRNA
  - WGCNA
  - hdWGCNA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# spatial-transcriptomics-co-expression-network-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs co-expression module discovery on spatial transcriptomics data using SGCRNA, a Julia spectral-clustering pipeline that avoids WGCNA's scale-free-topology assumption, and benchmarks module resolution against hdWGCNA (e.g. 114 SGCRNA modules vs hdWGCNA's two modules on the GSE261545 Visium kidney dataset).

## When to use

Use when analysing normalised spatial transcriptomics matrices (e.g. Visium GSE261545) for cell type-specific co-expression modules, or when WGCNA/hdWGCNA over-merges modules (grey/turquoise) and finer spatial module patterns are needed.

## When NOT to use

- The analysis goal requires the scale-free topology assumption or hub-gene ranking within classical WGCNA's framework — SGCRNA explicitly discards scale-free constraints
- Data is bulk RNA-seq with known trait association needs handled fine by standard WGCNA/PyWGCNA and no spatial or single-cell structure to exploit
- Only a coarse two-module (grey/turquoise) partition is desired; SGCRNA's higher module resolution would over-split

## Inputs

- Normalised spatial gene expression matrix (e.g. Seurat-processed Visium counts, GSE261545 kidney dataset)
- Gene symbol column (Data.Symbol) accompanying the expression matrix
- Tissue-spot / sample annotation table (e.g. SraRunTable.csv) for module-phenotype correlation

## Outputs

- Module-count comparison table (module_comparison_sgcrna_hdwgcna.csv: SGCRNA vs hdWGCNA)
- Per-module spatial enrichment assignments (e.g. tumour, proximal tubule, glomeruli, LH-CD, artery)
- CorPhenMod module-phenomenon correlation plot (Result/Fig/CorPhenMod.png)
- Network graph visualisation (DrawNetwork output, e.g. AllNetWork-0.5.png)

## How to apply

Normalise the spatial data with the standard Seurat spatial-transcriptomics workflow, then run SGCRNA in Julia: preprocess with CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST) to obtain the CorData correlation matrix and GradData regression-coefficient matrix, then cluster with SpectralClustering(CorData, GradData) using defaults (tNodeNum=100, depthMax=5). Run hdWGCNA on the same normalised expression matrix as the comparison baseline. Tabulate module counts per method, then correlate each module with tissue-spot annotations via CorPhenMod, ensuring sample/column order matches between the expression matrix and the annotation table (SraRunTable-style CSV). Reproduction succeeds if the reported pattern holds: hdWGCNA yields two modules (grey=tumour, turquoise=proximal tubule) while SGCRNA yields ~114 modules, ~71% non-cell-type-specific and the rest specific to tumour, healthy proximal tubules, glomeruli, LH-CD, and artery regions.

## Related tools

- **SGCRNA** (Core Julia package: CGM preprocessing (mode=:FTEST), SpectralClustering module discovery, SetNetwork/DrawNetwork visualisation, CorPhenMod module-phenotype correlation) — https://github.com/C37H41N2O6/SGCRNA
- **Julia 1.11** (Runtime environment; install SGCRNAs via Pkg.add("SGCRNAs"))
- **hdWGCNA** (Benchmark WGCNA variant optimised for single-cell and spatial transcriptomic data, run on the same normalised matrix for module-count comparison)
- **WGCNA** (Baseline co-expression method whose scale-free assumption and parameter tuning SGCRNA is designed to avoid)

## Examples

```
using SGCRNAs; CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST); clust, pos, edge_data = SpectralClustering(CorData, GradData);
```

## Evaluation signals

- hdWGCNA returns exactly two modules (grey and turquoise) on the Visium kidney data while SGCRNA partitions into ~114 modules
- ~71% of SGCRNA modules are non-cell-type-specific; remaining modules map to tumour, healthy proximal tubules, glomeruli, LH-CD, and artery regions
- hdWGCNA grey module enriches in tumour spots and turquoise module in proximal tubule spots
- CGM with mode=:FTEST writes a diagnostic plot with the range of values to be removed highlighted in red
- Sample/column order of the annotation table matches the expression matrix columns before CorPhenMod (verified via innerjoin on Symbol with order=:right)

## Limitations

- Conventional WGCNA's three principal limitations (scale-free assumption, manual parameter tuning, neglect of regression line slopes) motivated SGCRNA, but SGCRNA module counts may substantially exceed biologically interpretable groups (~29% of 114 modules cell type-specific on the kidney data)
- hdWGCNA may fail to resolve spatial heterogeneity, returning only two modules on the same input
- No changelog found for the repository, so version-to-version behavioural drift cannot be tracked from docs

## Evidence
