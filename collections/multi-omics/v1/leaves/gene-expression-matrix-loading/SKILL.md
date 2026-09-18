---
name: gene-expression-matrix-loading
description: Use when the input is a tab-delimited normalized count matrix (e.g. normalizedCounts_coding.tsv
  from GSE114007) that must be ingested in Julia before SGCRNA's CGM preprocessing
  and spectral clustering.
when_to_use_negative:
- Input is already an SGCRNA CGM output (correlation/regression-coefficient matrices)
  rather than a raw normalized-count TSV
- The matrix is stored in a non-tabular format (e.g. HDF5/AnnData) that CSV.jl cannot
  parse directly without prior conversion
- The analysis targets a different language stack (e.g. R/WGCNA), where the matrix
  would be read with R's own readers instead of CSV.jl/DataFrames.jl
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Julia
  role: Runtime language in which the expression matrix is loaded and passed to SGCRNA
    functions
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: SGCRNA (SGCRNAs.jl)
  role: Downstream package whose CGM/SpectralClustering functions consume the loaded
    matrix; README documents the loading call
  repo: https://github.com/C37H41N2O6/SGCRNA
merged_aliases:
- gene-expression-matrix-preprocessing
merged_alias_records:
- alias: gene-expression-matrix-preprocessing
  slug: gene-expression-matrix-preprocessing
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:619bf7b4534ab46b7d539ff8cb6e58db6493f7ffae88a343d2ca4f051c247e86
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
  - SGCRNA (SGCRNAs.jl)
  - SGCRNAs
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.2857142857142857
n_claims: 7
---

# gene-expression-matrix-loading

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Loads a TSV RNA expression matrix (variables as rows, samples as columns) into Julia using CSV.jl/DataFrames.jl as the entry point for SGCRNA co-expression analysis on datasets such as GSE114007.

## When to use

Use when the input is a tab-delimited normalized count matrix (e.g. normalizedCounts_coding.tsv from GSE114007) that must be ingested in Julia before SGCRNA's CGM preprocessing and spectral clustering.

## When NOT to use

- Input is already an SGCRNA CGM output (correlation/regression-coefficient matrices) rather than a raw normalized-count TSV
- The matrix is stored in a non-tabular format (e.g. HDF5/AnnData) that CSV.jl cannot parse directly without prior conversion
- The analysis targets a different language stack (e.g. R/WGCNA), where the matrix would be read with R's own readers instead of CSV.jl/DataFrames.jl

## Inputs

- Tab-delimited expression matrix TSV (e.g. assets/normalizedCounts_coding.tsv from GSE114007) with variables as rows and samples as columns
- Optional comma-delimited clinical data CSV (assets/SraRunTable.csv) for module-phenotype correlation

## Outputs

- DataFrames.jl DataFrame of normalized counts with a :Symbol gene-identifier column and numeric sample columns
- Matrix(Data[:,5:end]) numeric sample matrix ready for SGCRNAs.CGM preprocessing

## How to apply

Read the expression matrix with CSV.jl's CSV.read into a DataFrames.jl DataFrame, using header=1, comment='#', and delim='\t' as shown in the SGCRNAs.jl README, so gene symbols (Data.Symbol) are preserved alongside the numeric columns. Keep variables as rows and samples as columns; in the GSE114007 coding-gene matrix the numeric sample columns start at column 5, which is why CGM is called with Matrix(Data[:,5:end]). If clinical/phenotype data are needed downstream, load SraRunTable.csv separately (comma-delimited) and later join it back on :Symbol with innerjoin(..., order=:right). Verify the loaded frame has one row per gene and that the count of sample columns matches the SraRunTable samples before proceeding to CGM with mode=:FTEST/:SIGMA/:LESS filtering.

## Related tools

- **Julia** (Runtime language in which the expression matrix is loaded and passed to SGCRNA functions) — https://github.com/C37H41N2O6/SGCRNA
- **SGCRNA (SGCRNAs.jl)** (Downstream package whose CGM/SpectralClustering functions consume the loaded matrix; README documents the loading call) — https://github.com/C37H41N2O6/SGCRNA

## Examples

```
using CSV, DataFrames; Data = CSV.read("normalizedCounts_coding.tsv", header=1, comment="#", delim='\t', DataFrame); CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST);
```

## Evaluation signals

- CSV.read succeeds with header=1, comment='#', delim='\t' and returns a DataFrame whose first column is :Symbol
- Numeric sample columns begin at column 5, matching Matrix(Data[:,5:end]) passed to CGM in the README workflow
- Row count equals the number of coding genes in normalizedCounts_coding.tsv and no rows are dropped by the '#' comment filter
- The SraRunTable.csv sample metadata can be innerjoined back onto :Symbol with order=:right, confirming gene/sample identifiers are consistent
- Downstream CGM (mode=:FTEST) runs without dimension errors, implying the matrix orientation (variables as rows, samples as columns) is correct

## Limitations

- The README's column slicing (Data[:,5:end]) assumes exactly the GSE114007 coding-gene file layout (Symbol plus 4 leading non-sample columns); other matrices need their own column offset
- The load step alone performs no filtering — low-information variables are only removed when CGM is run with mode=:LESS, :SIGMA, or :FTEST
- Requires Julia 1.11 or later with the SGCRNAs package installed via Pkg.add("SGCRNAs")

## Evidence

- [readme] Load Data
Data = CSV.read("test_data/normalizedCounts_coding.tsv", header=1, comment="#", delim='\t', DataFrame);: [evidence span withheld — anchor status: not_found]
