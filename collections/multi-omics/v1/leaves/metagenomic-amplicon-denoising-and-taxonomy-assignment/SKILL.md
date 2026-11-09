---
name: metagenomic-amplicon-denoising-and-taxonomy-assignment
description: Use when raw 16S amplicon reads (e.g. from an SRA accession like PRJNA762466)
  must be denoised, feature-classified, and collapsed to genus level into a count
  matrix that will feed co-expression network methods such as SGCRNA or WGCNA.
when_to_use_negative:
- The input is already a genus-level count matrix or feature table — skip straight
  to SGCRNA's CGM preprocessing rather than re-running denoising/taxonomy assignment
- The data are bulk or single-cell RNA-seq gene expression matrices (e.g. normalizedCounts_coding.tsv
  with a Symbol column), for which the SGCRNA package's own example workflow applies
  directly without any amplicon/QIIME 2 step
- A scale-free-topology-constrained, hierarchical-clustering co-expression analysis
  is explicitly required and WGCNA's parameter-tuning workflow is acceptable
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: QIIME 2
  role: Runs denoising, feature classification, and taxonomy collapsing on the PRJNA762466
    amplicon reads to build the genus-level count matrix
- name: SGCRNAs (SGCRNA)
  role: Consumes the genus count matrix via CGM preprocessing (filter modes :LESS/:SIGMA/:FTEST)
    and SpectralClustering to partition genera into modules
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: WGCNA
  role: Comparative baseline run on the same genus count matrix to yield module counts
    for benchmarking (8 modules vs SGCRNA's 12)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:4781f468f3b4af2932927b2a8ed35ac6208ecc314dda61c48edb98cbee7f60da
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
  - QIIME 2
  - SGCRNAs
  - SGCRNAs (SGCRNA)
  - WGCNA
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# metagenomic-amplicon-denoising-and-taxonomy-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A QIIME 2-based pipeline that takes raw 16S amplicon sequencing reads (e.g. PRJNA762466 from SRA) through denoising, feature classification, and taxonomy collapsing to produce a genus-level count matrix (variables as rows, samples as columns) suitable for downstream co-expression network analyses such as SGCRNA and WGCNA.

## When to use

Use when raw 16S amplicon reads (e.g. from an SRA accession like PRJNA762466) must be denoised, feature-classified, and collapsed to genus level into a count matrix that will feed co-expression network methods such as SGCRNA or WGCNA.

## When NOT to use

- The input is already a genus-level count matrix or feature table — skip straight to SGCRNA's CGM preprocessing rather than re-running denoising/taxonomy assignment
- The data are bulk or single-cell RNA-seq gene expression matrices (e.g. normalizedCounts_coding.tsv with a Symbol column), for which the SGCRNA package's own example workflow applies directly without any amplicon/QIIME 2 step
- A scale-free-topology-constrained, hierarchical-clustering co-expression analysis is explicitly required and WGCNA's parameter-tuning workflow is acceptable

## Inputs

- 16S amplicon sequencing reads from SRA (e.g. accession PRJNA762466)
- QIIME 2 denoising/feature-classification/taxonomy-collapsing pipeline parameters
- Optional: SraRunTable.csv clinical/metadata table (used downstream for module-phenomenon correlation)

## Outputs

- Genus-level count matrix with variables as rows and samples as columns (208 genera before preprocessing)
- Diagnostic plot saved by CGM in the current directory, with the range of removed values highlighted in red
- SGCRNA clustering results stored in `clust` (12 modules on the retained 15 genera)
- WGCNA module assignment (8 modules) as comparative baseline
- sgcrna_wgcna_module_counts.tsv compiling retention count and module counts

## How to apply

Download the amplicon sequencing reads for the target accession (e.g. PRJNA762466) from SRA and run the QIIME 2 pipeline covering denoising, feature classification, and taxonomy collapsing to obtain a genus-level count matrix with variables (genera) as rows and samples as columns. This matrix is the required input format for SGCRNA's CGM preprocessing function, which is called on the matrix using one of its filter modes (:LESS, :SIGMA, or :FTEST — default :NONE) and additionally saves a diagnostic plot with the range of values to be removed highlighted in red. For the PRJNA762466 benchmark, CGM retained 15 of 208 genera, after which SGCRNA's SpectralClustering function on the CGM correlation and regression coefficient matrices yielded 12 modules versus 8 modules for WGCNA run as the comparative baseline. Compile the retention count and module counts into a TSV (sgcrna_wgcna_module_counts.tsv) for validation.

## Related tools

- **QIIME 2** (Runs denoising, feature classification, and taxonomy collapsing on the PRJNA762466 amplicon reads to build the genus-level count matrix)
- **SGCRNAs (SGCRNA)** (Consumes the genus count matrix via CGM preprocessing (filter modes :LESS/:SIGMA/:FTEST) and SpectralClustering to partition genera into modules) — https://github.com/C37H41N2O6/SGCRNA
- **WGCNA** (Comparative baseline run on the same genus count matrix to yield module counts for benchmarking (8 modules vs SGCRNA's 12))

## Evaluation signals

- The QIIME 2 output is a genus-level count matrix oriented with variables as rows and samples as columns, ready for CGM(Data.Symbol, Matrix(Data[:,5:end]))-style consumption
- Reproduction check: 15 of 208 genera are retained after SGCRNA pre-processing on PRJNA762466
- Reproduction check: SGCRNA yields 12 modules and WGCNA yields 8 modules on the same matrix
- CGM with a filter mode specified saves a diagnostic plot in the current directory with the removed value range highlighted in red
- Compiled results table sgcrna_wgcna_module_counts.tsv contains exactly the reported retention and module-count values

## Limitations

- The task card specifies one of :LESS, :SIGMA, or :FTEST as the CGM filter mode but does not record which specific mode was used for the PRJNA762466 run; the mode choice affects which of the 208 genera are retained
- SGCRNAs.jl requires Julia 1.11 or later and is installed as a Julia package, so the downstream analysis cannot be run purely in R/Python (PyWGCNA exists, but SGCRNA itself is Julia-only)
- The paper's concrete worked example and test data (normalizedCounts_coding.tsv, SraRunTable.csv) are RNA expression data, so amplicon users must adapt the genus count matrix to the same variables-as-rows layout
- No changelog was found for the SGCRNAs.jl repository, so version-specific behavioral changes are not traceable

## Evidence

- [other] Download the PRJNA762466 amplicon sequencing reads from SRA and run the QIIME 2 pipeline (denoising, feature classification, taxonomy collapsing): [evidence span withheld — anchor status: not_found]
