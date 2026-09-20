---
name: wgcna-scale-free-parameter-tuning
description: Use when a co-expression analysis must assume a scale-free topology and
  needs the R WGCNA package run with manually tuned parameters (soft-thresholding
  power) on a gene-by-sample count matrix, e.g. as a baseline for comparison with
  scale-free-free methods like SGCRNA.
when_to_use_negative:
- When the dataset's topology cannot be validated as scale-free, or when you want
  to avoid manual parameter tuning entirely — SGCRNA was proposed precisely because
  the scale-free assumption is less prevalent than previously assumed and WGCNA's
  second parameter requires considerable effort to optimise
- When correlation coefficients alone would be misleading (genes with minimal expression
  variation but high correlation may falsely imply co-expression) — regression-aware
  methods like SGCRNA's CGM address this
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: WGCNA
  role: R co-expression network analysis package whose essential parameters (including
    soft-thresholding power) are manually tuned to satisfy the scale-free topology
    criterion, yielding the baseline module count
- name: SGCRNAs
  role: Julia counterpart method (CGM preprocessing + SpectralClustering with default
    tNodeNum=100, depthMax=5) whose module count (76 on GSE114007) provides the scale-free-free
    comparison baseline
  repo: https://github.com/C37H41N2O6/SGCRNA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:7543f8310f17d1535ff7455997be3fbf14d637c5cb5ecdb112d5af59de7e1368
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
  - SGCRNAs
  - WGCNA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# wgcna-scale-free-parameter-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run the R WGCNA package on a bulk RNA-Seq expression matrix (e.g. GSE114007 normalizedCounts_coding.tsv) while manually tuning the essential parameters, including the soft-thresholding power, so that the network satisfies the scale-free topology criterion — serving as the conventional co-expression baseline that SGCRNA is compared against.

## When to use

Use when a co-expression analysis must assume a scale-free topology and needs the R WGCNA package run with manually tuned parameters (soft-thresholding power) on a gene-by-sample count matrix, e.g. as a baseline for comparison with scale-free-free methods like SGCRNA.

## When NOT to use

- When the dataset's topology cannot be validated as scale-free, or when you want to avoid manual parameter tuning entirely — SGCRNA was proposed precisely because the scale-free assumption is less prevalent than previously assumed and WGCNA's second parameter requires considerable effort to optimise
- When correlation coefficients alone would be misleading (genes with minimal expression variation but high correlation may falsely imply co-expression) — regression-aware methods like SGCRNA's CGM address this

## Inputs

- Normalized bulk RNA-Seq expression matrix with variables as rows and samples as columns (TSV, e.g. normalizedCounts_coding.tsv from GSE114007)
- Optional clinical/phenotype data table (e.g. SraRunTable.csv) for downstream module-trait association

## Outputs

- Per-gene WGCNA module labels
- Total module count under scale-free-validating parameters (reported: 14 on GSE114007)
- module_counts.csv (per-method module counts)
- module_assignments.csv (module-gene assignments)

## How to apply

Run the R WGCNA package on the same expression matrix used by the comparison method (variables as rows, samples as columns, e.g. the normalizedCounts_coding.tsv matrix from GSE114007). Manually determine the two essential WGCNA parameters — in particular the soft-thresholding power — by tuning until the scale-free topology criterion is satisfied, since WGCNA relies on this assumption and requires considerable effort to optimise the second parameter. Record the resulting module count (reported: 14 modules on GSE114007 vs 76 for SGCRNA) and the per-gene module labels. Tabulate per-method module counts and module-gene assignments (module_counts.csv, module_assignments.csv) and validate that the module count reproduces the published value.

## Related tools

- **WGCNA** (R co-expression network analysis package whose essential parameters (including soft-thresholding power) are manually tuned to satisfy the scale-free topology criterion, yielding the baseline module count)
- **SGCRNAs** (Julia counterpart method (CGM preprocessing + SpectralClustering with default tNodeNum=100, depthMax=5) whose module count (76 on GSE114007) provides the scale-free-free comparison baseline) — https://github.com/C37H41N2O6/SGCRNA

## Examples

```
using SGCRNAs; CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST); clust, pos, edge_data = SpectralClustering(CorData, GradData);  # then run R WGCNA with tuned soft-thresholding power on the same matrix
```

## Evaluation signals

- Recorded WGCNA module count on GSE114007 reproduces the published value of 14 modules (vs 76 for SGCRNA)
- The chosen soft-thresholding power and other essential parameters demonstrably satisfy the scale-free topology criterion
- module_counts.csv and module_assignments.csv are produced with complete per-gene assignments matching the expression matrix row names
- Per-gene module labels can be diffed against SGCRNA's SpectralClustering labels to quantify the difference in partition granularity

## Limitations

- WGCNA assumes a scale-free topology, which recent studies suggest is less prevalent than previously assumed
- Requires complex manual parameter tuning, with the second parameter in particular requiring considerable effort to optimise
- WGCNA disregards regression coefficients, so a gene with minimal expression variation but high correlation coefficient may falsely imply co-expression

## Evidence

- [other] baseline module counts on GSE114007: [evidence span withheld — anchor status: not_found]
- [readme] SGCRNAs Julia usage on the same matrix: [evidence span withheld — anchor status: not_found]
