---
name: co-expression-module-gene-list-extraction
description: Use when a co-expression network has been clustered into modules (e.g.
  SGCRNA SpectralClustering output or WGCNA brown4/grey assignments) and per-module
  gene lists are needed for downstream enrichment or cross-module comparison.
when_to_use_negative:
- The co-expression network has not yet been clustered — no clust vector or WGCNA
  module assignments exist to extract genes from.
- Gene identifiers in the expression matrix and module assignments differ (no Symbol
  join key), so extracted lists would not map to a consistent gene space.
- Only a single gene list is needed and no module structure comparison (SGCRNA vs
  WGCNA) is intended.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SGCRNA
  role: Julia package whose CGM preprocessing and SpectralClustering functions produce
    the module (cluster) assignments from which gene lists are extracted
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: Julia
  role: Execution environment for SGCRNAs.jl functions and CSV/DataFrames handling
    of the expression matrix
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: WGCNA
  role: Alternative co-expression method whose module assignments (brown4, grey) provide
    comparison gene lists alongside the SGCRNA modules
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:5a8d3bb0f2c9d00870a1b90b682206b76d6943349aad0e134f4e02283ab22358
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
  - SGCRNA
  - WGCNA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# co-expression-module-gene-list-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts the gene members of co-expression modules produced by SGCRNA's spectral clustering (or WGCNA) so the per-module gene lists can feed downstream analyses such as Gene Ontology enrichment. In the SGCRNA workflow this means collecting gene Symbols from the SGCRNAs.jl CGM/SpectralClustering output and the WGCNA module assignment tables.

## When to use

Use when a co-expression network has been clustered into modules (e.g. SGCRNA SpectralClustering output or WGCNA brown4/grey assignments) and per-module gene lists are needed for downstream enrichment or cross-module comparison.

## When NOT to use

- The co-expression network has not yet been clustered — no clust vector or WGCNA module assignments exist to extract genes from.
- Gene identifiers in the expression matrix and module assignments differ (no Symbol join key), so extracted lists would not map to a consistent gene space.
- Only a single gene list is needed and no module structure comparison (SGCRNA vs WGCNA) is intended.

## Inputs

- Normalized RNA expression matrix (TSV with Symbol column, e.g. normalizedCounts_coding.tsv)
- SGCRNAs.jl CGM/SpectralClustering output (clust vector, edge_data)
- WGCNA module assignment table (e.g. brown4, grey modules) for cross-method comparison

## Outputs

- Per-module gene lists (gene Symbols) for each SGCRNA and WGCNA module
- Combined module-gene assignment table (e.g. TSV) suitable as input to GO enrichment

## How to apply

Load the expression matrix (e.g. normalizedCounts_coding.tsv, tab-delimited with a Symbol column) and run SGCRNAs.jl: preprocess with CGM (optionally with mode=:FTEST to filter noisy genes), then call SpectralClustering to obtain cluster assignments. From the resulting clust vector (and the joined Data frame whose rows were matched to edge_data via innerjoin on :Symbol), index each module ID (e.g. modules 45 and 70 from the GSE114007 analysis) to pull its gene Symbols. Do the same for comparison modules from the WGCNA assignment tables (brown4, grey). Validate the extracted lists by checking that module sizes are non-zero, that Symbols are unique and map back to the expression matrix rows, and that cross-method module comparisons use consistent gene identifier spaces.

## Related tools

- **SGCRNA** (Julia package whose CGM preprocessing and SpectralClustering functions produce the module (cluster) assignments from which gene lists are extracted) — https://github.com/C37H41N2O6/SGCRNA
- **Julia** (Execution environment for SGCRNAs.jl functions and CSV/DataFrames handling of the expression matrix) — https://github.com/C37H41N2O6/SGCRNA
- **WGCNA** (Alternative co-expression method whose module assignments (brown4, grey) provide comparison gene lists alongside the SGCRNA modules)

## Examples

```
using CSV, DataFrames, SGCRNAs; Data = CSV.read("normalizedCounts_coding.tsv", header=1, delim='\t', DataFrame); CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST); clust, pos, edge_data = SpectralClustering(CorData, GradData);
```

## Evaluation signals

- Each extracted module gene list is non-empty and contains unique gene Symbols drawn from the Symbol column of the expression matrix.
- Genes in each list belong to exactly one module ID in the clust vector (module memberships are disjoint).
- Every extracted Symbol matches a row of edge_data after the innerjoin on :Symbol, confirming identifier consistency.
- Cross-method lists (SGCRNA modules 45/70 vs WGCNA brown4/grey) are comparable in identifier space, and GO enrichment of the lists reproduces reported associations (module 45 = ER, module 70 = ECM genes such as MMP11, MMP13, ADAMTS7).

## Limitations

- CGM mode (:NONE, :LESS, :SIGMA, :FTEST) changes which genes are removed before clustering, so extracted lists depend on the chosen filter mode and its saved plot highlighting removed value ranges.
- Module numbering depends on the SpectralClustering run; module IDs (e.g. 45, 70) are not stable across parameter changes or different depth selections (d in SetNetwork).

## Evidence

- [abstract] SGCRNA provides Julia functions for the analysis of co-expression networks derived from various types of biological data, such as gene expression data.: "SGCRNA provides Julia functions for the analysis of co-expression networks derived from various types of biological data, such as gene expression data."
- [methods] An optional argument, `mode` (default: `NONE`), can be set to `:LESS`, `:SIGMA`, or `:FTEST`.: "An optional argument, `mode` (default: `NONE`), can be set to `:LESS`, `:SIGMA`, or `:FTEST`."
- [methods] Clustering is performed using the results obtained in the previous step.: "Clustering is performed using the results obtained in the previous step."
- [readme] CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST); clust, pos, edge_data = SpectralClustering(CorData, GradData);: [evidence span withheld — anchor status: not_found]
