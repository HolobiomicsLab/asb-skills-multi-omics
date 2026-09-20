---
name: genus-level-feature-table-collapsing
description: Use when 16S amplicon sequencing reads (e.g. PRJNA762466 from SRA) must
  be denoised, classified, and collapsed to genus level with QIIME 2 before co-expression
  module detection with SGCRNA or WGCNA.
when_to_use_negative:
- The input is already a collapsed genus-level feature table — go directly to CGM/SpectralClustering
  rather than re-running QIIME 2 collapsing.
- The data are RNA gene-expression counts (e.g. GSE114007 normalizedCounts_coding.tsv)
  rather than 16S amplicon data — no taxonomy collapsing step is needed.
- Only raw uncollapsed ASV/feature-level data are required downstream, and genus-level
  aggregation would discard taxonomic resolution the analysis depends on.
edam_operation: http://edamontology.org/operation_3432
tools:
- name: QIIME 2
  role: Runs denoising, feature classification, and taxonomy collapsing on SRA amplicon
    reads to produce the genus-level count matrix
- name: SGCRNAs
  role: Consumes the collapsed genus-level matrix via CGM preprocessing (mode :LESS/:SIGMA/:FTEST)
    and SpectralClustering to derive modules
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: Julia
  role: Runtime environment (1.11 or later) in which SGCRNAs functions are executed
    on the collapsed table
- name: WGCNA
  role: Comparative baseline module detection run on the same collapsed genus-level
    matrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:32cc5ed9eeb026b7565b61c0f1c60e69db1c8011835f8cfdb6ee7a2ae17838ab
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# genus-level-feature-table-collapsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepares a genus-level count matrix from 16S amplicon data (e.g. PRJNA762466) via the QIIME 2 pipeline — denoising, feature classification, and taxonomy collapsing — formatted with variables (genera) as rows and samples as columns, so it can feed co-expression network methods such as SGCRNA's CGM/SpectralClustering and WGCNA.

## When to use

Use when 16S amplicon sequencing reads (e.g. PRJNA762466 from SRA) must be denoised, classified, and collapsed to genus level with QIIME 2 before co-expression module detection with SGCRNA or WGCNA.

## When NOT to use

- The input is already a collapsed genus-level feature table — go directly to CGM/SpectralClustering rather than re-running QIIME 2 collapsing.
- The data are RNA gene-expression counts (e.g. GSE114007 normalizedCounts_coding.tsv) rather than 16S amplicon data — no taxonomy collapsing step is needed.
- Only raw uncollapsed ASV/feature-level data are required downstream, and genus-level aggregation would discard taxonomic resolution the analysis depends on.

## Inputs

- 16S amplicon sequencing reads from SRA (e.g. PRJNA762466, FASTQ)
- QIIME 2 feature classification / taxonomy artifacts (intermediate)
- genus-level count matrix TSV with variables as rows and samples as columns

## Outputs

- genus-level count matrix (TSV; rows = genera, columns = samples)
- SGCRNAs CGM diagnostic plot with removed value ranges highlighted in red
- module assignments from SGCRNAs.SpectralClustering (clust) and WGCNA
- comparison table sgcrna_wgcna_module_counts.tsv (retention count and module counts)

## How to apply

Download the amplicon reads from SRA (e.g. PRJNA762466) and run the QIIME 2 pipeline — denoising, feature classification, and taxonomy collapsing — to obtain a genus-level count matrix with variables as rows and samples as columns. Load the resulting TSV in Julia (CSV.read with header=1, comment='#', delim='\t') so genus symbols form the row keys. Pass the matrix to SGCRNAs.CGM with a filter mode (:LESS, :SIGMA, or :FTEST); specifying a mode also saves a diagnostic plot in the current directory with the range of removed values highlighted in red. On the PRJNA762466 validation case, CGM retained 15 of 208 genera, SGCRNAs.SpectralClustering on the CGM correlation and regression coefficient outputs yielded 12 modules, and WGCNA on the same collapsed matrix yielded 8 modules. Compile the retention count and both module counts into sgcrna_wgcna_module_counts.tsv for comparison.

## Related tools

- **QIIME 2** (Runs denoising, feature classification, and taxonomy collapsing on SRA amplicon reads to produce the genus-level count matrix)
- **SGCRNAs** (Consumes the collapsed genus-level matrix via CGM preprocessing (mode :LESS/:SIGMA/:FTEST) and SpectralClustering to derive modules) — https://github.com/C37H41N2O6/SGCRNA
- **Julia** (Runtime environment (1.11 or later) in which SGCRNAs functions are executed on the collapsed table)
- **WGCNA** (Comparative baseline module detection run on the same collapsed genus-level matrix)

## Examples

```
Data = CSV.read("genus_table.tsv", header=1, comment="#", delim='\t', DataFrame); CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST); clust, pos, edge_data = SpectralClustering(CorData, GradData);
```

## Evaluation signals

- After CGM preprocessing on the PRJNA762466 genus matrix, exactly 15 of 208 genera are retained for network analysis
- SGCRNAs.SpectralClustering yields 12 modules on the retained genera; WGCNA on the same matrix yields 8 modules
- The collapsed count matrix has variables as rows and samples as columns, loadable via CSV.read(..., header=1, comment='#', delim='\t', DataFrame)
- A diagnostic plot is saved in the current directory with the range of values to be removed highlighted in red when a filter mode is set
- The final sgcrna_wgcna_module_counts.tsv contains the retention count (15/208) and both module counts (12 SGCRNA, 8 WGCNA)

## Limitations

- The CGM filter mode is optional (default :NONE) and the choice among :LESS, :SIGMA, and :FTEST changes which genera are retained, so results depend on this decision point
- SGCRNA's Julia functions were demonstrated on gene expression data (GSE114007); the 16S genus-level application is a comparative use case rather than the package's primary example
- Only 15 of 208 genera survive preprocessing on this dataset, leaving a small network for module detection

## Evidence

- [other] run the QIIME 2 pipeline (denoising, feature classification, taxonomy collapsing) to obtain a genus-level count matrix with variables as rows and samples as columns: [evidence span withheld — anchor status: not_found]
- [other] After SGCRNA pre-processing, 15 genera were included in the network analysis; SGCRNA partitioned these genera into 12 modules, whereas WGCNA yielded only 8.: [evidence span withheld — anchor status: not_found]
- [readme] Data = CSV.read("test_data/normalizedCounts_coding.tsv", header=1, comment="#", delim='\t', DataFrame);: [evidence span withheld — anchor status: not_found]
