---
name: transcriptomic-dataset-handling
description: Use when a tab-separated RNA expression matrix (e.g. assets/normalizedCounts_coding.tsv
  with gene symbols) and clinical data (e.g. assets/SraRunTable.csv) must be read,
  cleaned, and aligned by sample before clustering or module–phenotype correlation.
when_to_use_negative:
- The input is already a preprocessed, sample-aligned module assignment table (e.g.
  SGCRNA or WGCNA module gene lists awaiting GO enrichment)
- The expression data are single-cell (the article only demonstrates this on bulk
  RNA normalized counts from GSE114007)
- Phenotype metadata are missing or sample identifiers cannot be matched to expression
  columns, since module–phenotype correlation requires the inner join on :Symbol
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SGCRNAs
  role: Provides the CGM preprocessing function and the Julia functions whose edge_data
    gene set defines which counts are retained and joined
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: Julia
  role: Runtime environment (Julia 1.11 or later) with CSV/DataFrames used to read
    TSV and CSV inputs and perform the sample-aligned inner join
- name: WGCNA
  role: Comparison co-expression pipeline whose module assignments (e.g. brown4, grey
    on GSE114007) consume the same handled transcriptomic inputs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:0132d44d4c6dc5815b2d1c931db971d2b895191715de1366184f78adf650869a
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.125
n_claims: 8
---

# transcriptomic-dataset-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Loads and harmonises transcriptomic count matrices and clinical metadata for co-expression analysis in Julia, as done in the SGCRNA workflow on the GSE114007 osteoarthritis dataset (normalizedCounts_coding.tsv plus SraRunTable.csv).

## When to use

Use when a tab-separated RNA expression matrix (e.g. assets/normalizedCounts_coding.tsv with gene symbols) and clinical data (e.g. assets/SraRunTable.csv) must be read, cleaned, and aligned by sample before clustering or module–phenotype correlation.

## When NOT to use

- The input is already a preprocessed, sample-aligned module assignment table (e.g. SGCRNA or WGCNA module gene lists awaiting GO enrichment)
- The expression data are single-cell (the article only demonstrates this on bulk RNA normalized counts from GSE114007)
- Phenotype metadata are missing or sample identifiers cannot be matched to expression columns, since module–phenotype correlation requires the inner join on :Symbol

## Inputs

- RNA expression matrix TSV (assets/normalizedCounts_coding.tsv) with a gene Symbol column and sample columns starting at column 5
- Clinical/phenotype metadata CSV (assets/SraRunTable.csv)
- GEO dataset identifier (GSE114007) defining the source samples

## Outputs

- Cleaned Julia DataFrame of normalised counts with symbols retained and low-quality values filtered via CGM (:FTEST/:SIGMA/:LESS)
- Sorted phenotype DataFrame aligned to expression samples
- Inner-joined expression table restricted to clustered genes (names(edge_data)) ready for CorPhenMod

## How to apply

Read the expression matrix with CSV.read into a Julia DataFrame using header=1, comment='#', delim='\t' so that gene symbols (Data.Symbol) are preserved as identifiers. Run the SGCRNAs CGM function on Data.Symbol and Matrix(Data[:,5:end]) for preprocessing, optionally with mode=:FTEST (alternatives :LESS or :SIGMA), which saves a plot with the range of values removed highlighted in red as a sanity check. Load the SraRunTable.csv phenotype file (comma-delimited) and sort it; then innerjoin the expression DataFrame on :Symbol against DataFrame(Symbol=names(edge_data)) ordered right, so only genes retained by clustering are kept and sample columns stay aligned to the phenotype rows. This makes CorPhenMod module–phenotype correlation (e.g. against Phen[:,[2,3,5]]) valid, as performed for the GSE114007 dataset.

## Related tools

- **SGCRNAs** (Provides the CGM preprocessing function and the Julia functions whose edge_data gene set defines which counts are retained and joined) — https://github.com/C37H41N2O6/SGCRNA
- **Julia** (Runtime environment (Julia 1.11 or later) with CSV/DataFrames used to read TSV and CSV inputs and perform the sample-aligned inner join)
- **WGCNA** (Comparison co-expression pipeline whose module assignments (e.g. brown4, grey on GSE114007) consume the same handled transcriptomic inputs)

## Examples

```
using CSV, DataFrames; Data = CSV.read("assets/normalizedCounts_coding.tsv", header=1, comment="#", delim='\t', DataFrame); Phen = CSV.read("assets/SraRunTable.csv", header=1, comment="#", delim=',', DataFrame); sort!(Phen);
```

## Evaluation signals

- CSV.read of the TSV succeeds with comment='#' stripping, yielding a DataFrame whose Symbol column and sample columns match the expected layout (expression starts at column 5)
- CGM with mode=:FTEST produces a saved plot in which the removed value range is highlighted in red, confirming filtering occurred as intended
- The innerjoin on :Symbol returns exactly the genes in names(edge_data), confirming only clustered genes survive and no sample misalignment
- Phenotype rows of SraRunTable.csv are sorted and index-matched to the joined expression columns before CorPhenMod is called
- Downstream module–phenotype correlation (CorPhenMod on Phen[:,[2,3,5]]) runs without dimension errors, indicating counts and clinical data are sample-consistent

## Limitations

- CGM filtering modes (:LESS, :SIGMA, :FTEST) silently remove values unless the user inspects the generated plot showing the removed range in red
- Data cleaning is interleaved with SGCRNA-specific structures (edge_data column ordering), so reordering genes before the innerjoin can break sample alignment
- The workflow assumes the expression matrix layout of normalizedCounts_coding.tsv (Symbol + samples from column 5); other layouts require index adjustments

## Evidence

- [readme] Data loading layout: [evidence span withheld — anchor status: not_found]
- [methods] Source dataset: [evidence span withheld — anchor status: not_found]
