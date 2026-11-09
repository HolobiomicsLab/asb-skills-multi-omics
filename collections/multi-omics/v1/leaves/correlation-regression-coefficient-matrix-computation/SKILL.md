---
name: correlation-regression-coefficient-matrix-computation
description: Use when preparing a normalized gene/variable expression matrix (genes
  as row identifiers, samples as columns, e.g. normalizedCounts_coding.tsv) for co-expression
  clustering, and you need correlation plus regression-slope matrices as input to
  spectral clustering.
when_to_use_negative:
- When the downstream method requires a scale-free-topology assumption or weighted
  adjacency as in WGCNA — this skill's rationale explicitly discards the scale-free
  assumption in favor of spectral clustering
- When only a single association measure is needed and regression slopes are irrelevant;
  the paired CorData/GradData output is designed so slopes guard against high-correlation/low-variation
  false co-expression
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3524
tools:
- name: SGCRNAs
  role: Provides the CGM Julia function that performs the pre-processing and returns
    CorData (correlation matrix) and GradData (regression coefficient matrix)
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: Julia
  role: Runtime language (1.11 or later recommended) in which SGCRNAs.jl is installed
    via Pkg.add and executed
- name: CSV / DataFrames
  role: Used in the README example to load the normalized count TSV into a DataFrame
    before extracting the Symbol column and sample matrix for CGM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:5341e0d17cda7b50013a0f4e966b3b5555f1f7e057ea6c639bd2e2f9c749735d
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
  - CSV / DataFrames
  - Julia
  - SGCRNAs
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# correlation-regression-coefficient-matrix-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes paired correlation (CorData) and regression-coefficient (GradData) matrices from an expression matrix via SGCRNAs.jl's CGM pre-processing function, capturing both association strength and regression line slope so genes with minimal expression variation are not falsely flagged as co-expressed.

## When to use

Use when preparing a normalized gene/variable expression matrix (genes as row identifiers, samples as columns, e.g. normalizedCounts_coding.tsv) for co-expression clustering, and you need correlation plus regression-slope matrices as input to spectral clustering.

## When NOT to use

- When the downstream method requires a scale-free-topology assumption or weighted adjacency as in WGCNA — this skill's rationale explicitly discards the scale-free assumption in favor of spectral clustering
- When only a single association measure is needed and regression slopes are irrelevant; the paired CorData/GradData output is designed so slopes guard against high-correlation/low-variation false co-expression

## Inputs

- Normalized expression matrix (TSV/DataFrame), variables as rows with a Symbol identifier column and samples as columns (e.g. normalizedCounts_coding.tsv, expression columns from column 5 onward)
- Vector of variable/gene symbols corresponding to matrix rows
- Optional filter mode argument: :NONE (default), :LESS, :SIGMA, or :FTEST

## Outputs

- CorData: correlation matrix (with variable symbols)
- GradData: regression coefficient matrix
- Optional frequency plot saved in the current directory with the removed value range highlighted in red (when mode is not :NONE)

## How to apply

Load the normalized count matrix (e.g. TSV with gene Symbol column and expression columns from column 5 onward) and call CGM with the gene symbol vector and the sample matrix. Optionally set mode to :LESS, :SIGMA, or :FTEST (default :NONE); any non-:NONE mode triggers generation of a frequency plot in the current directory with the range of values to be removed highlighted in red, excluding non-significant variables before clustering. The function returns two matrices: CorData (correlation matrix) and GradData (regression coefficient matrix); the slope component is the key rationale, because WGCNA-style correlation alone can falsely imply co-expression when a gene shows minimal variation in expression but a high correlation coefficient. Verify against ground truth that genes with near-zero expression variation or negligible regression slopes are excluded, and that the returned matrices carry gene symbols consistent with the retained set before passing them to SpectralClustering.

## Related tools

- **SGCRNAs** (Provides the CGM Julia function that performs the pre-processing and returns CorData (correlation matrix) and GradData (regression coefficient matrix)) — https://github.com/C37H41N2O6/SGCRNA
- **Julia** (Runtime language (1.11 or later recommended) in which SGCRNAs.jl is installed via Pkg.add and executed)
- **CSV / DataFrames** (Used in the README example to load the normalized count TSV into a DataFrame before extracting the Symbol column and sample matrix for CGM)

## Examples

```
using CSV, DataFrames, SGCRNAs; Data = CSV.read("normalizedCounts_coding.tsv", header=1, delim='\t', DataFrame); CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST);
```

## Evaluation signals

- CGM returns exactly two outputs, CorData and GradData, that can be fed directly into SpectralClustering(CorData, GradData)
- With mode set to :LESS, :SIGMA, or :FTEST, a frequency plot file is generated in the current directory with the value range to be removed highlighted in red
- Genes with near-zero expression variation or negligible regression slopes are excluded from the retained matrices
- Retained gene symbols in CorData/GradData match the input Symbol column ordering and the subset of non-removed variables
- For small synthetic test matrices, adjust tNodeNum and pcas below their defaults (100 and 99) to less than roughly one-tenth of the variable count (minimum 3) so the downstream clustering step runs

## Limitations

- Filtering behaviour differs by mode: :NONE (default) applies no removal, while :LESS, :SIGMA, and :FTEST remove non-significant variables — the mode choice changes the retained variable set
- Filter-mode selection relies on visual inspection of the generated frequency plot to confirm the removed value range is appropriate
- Matrix sizes downstream depend on internal parameters tNodeNum and pcas (defaults 100 and 99), which must be reduced for small variable counts

## Evidence

- [methods] CGM pre-processing with filter modes: "An optional argument, `mode` (default: `:NONE`), can be set to `:LESS`, `:SIGMA`, or `:FTEST`."
