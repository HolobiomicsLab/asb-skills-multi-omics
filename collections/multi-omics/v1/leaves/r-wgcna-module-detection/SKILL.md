---
name: r-wgcna-module-detection
description: Use when benchmarking WGCNA module detection against SGCRNA on the same
  expression matrix, or when reproducing the SGCRNA-vs-WGCNA execution-time comparison
  on GSE114007 (normalizedCounts_coding.tsv).
when_to_use_negative:
- The goal is to avoid the scale-free-topology assumption or manual power/cutHeight
  parameter tuning — use SGCRNA (SpectralClustering via SGCRNAs.jl) instead, since
  WGCNA relies on the scale-free assumption and requires complex manual parameter
  tuning.
- Genes with minimal expression variation matter for the analysis — the article notes
  WGCNA disregards regression coefficients, so a gene with minimal variation but high
  correlation may falsely imply co-expression; SGCRNA addresses this.
tools:
- name: WGCNA
  role: 'R package performing the module detection: soft-thresholding power=20, module
    merge cutHeight=0.9, timed including network/plot generation over 10 runs'
- name: SGCRNAs.jl
  role: Julia package providing the SGCRNA comparison baseline (CGM preprocessing
    with mode=:FTEST, SpectralClustering, SetNetwork, DrawNetwork) timed on the same
    GSE114007 matrix
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: Julia 1.11
  role: Runtime environment for the SGCRNA side of the timing comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:74cfd19129f798360a50c41666b9478559bda713c5e18bd026e2673ecee75633
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
  - SGCRNAs.jl
  - WGCNA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.2
n_claims: 5
---

# r-wgcna-module-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detect co-expression gene modules in R with WGCNA by soft-thresholding the correlation network (power=20), merging modules at cutHeight=0.9, and including network/plot generation in the measurement — here used as the runtime baseline against SGCRNA's spectral-clustering pipeline on GSE114007.

## When to use

Use when benchmarking WGCNA module detection against SGCRNA on the same expression matrix, or when reproducing the SGCRNA-vs-WGCNA execution-time comparison on GSE114007 (normalizedCounts_coding.tsv).

## When NOT to use

- The goal is to avoid the scale-free-topology assumption or manual power/cutHeight parameter tuning — use SGCRNA (SpectralClustering via SGCRNAs.jl) instead, since WGCNA relies on the scale-free assumption and requires complex manual parameter tuning.
- Genes with minimal expression variation matter for the analysis — the article notes WGCNA disregards regression coefficients, so a gene with minimal variation but high correlation may falsely imply co-expression; SGCRNA addresses this.

## Inputs

- GSE114007 RNA expression matrix (normalizedCounts_coding.tsv, TSV, genes as rows / samples as columns)
- GSE114007 clinical data (assets/SraRunTable.csv, CSV)

## Outputs

- timing_comparison.csv with mean and standard deviation of 10 WGCNA runtimes vs 10 SGCRNA runtimes
- WGCNA module assignments, network plots, and dendrograms generated within each timed run

## How to apply

Load the GSE114007 RNA expression matrix (normalizedCounts_coding.tsv, variables as rows and samples as columns) and run WGCNA in R 10 times, timing each full run with wall-clock measurement. In each run apply soft-thresholding power=20 and merge modules at cutHeight=0.9, including the time to build the network and generate plots in each timing measurement. Aggregate the 10 runtimes into a mean and standard deviation per method and write them to timing_comparison.csv alongside the SGCRNA timings. The rationale for the comparison is that SGCRNA's spectral clustering computes eigenvectors, which the article contrasts with WGCNA's scale-free-topology assumption and manual parameter tuning; the expected outcome is that mean WGCNA runtime is substantially (markedly, not equal within noise) lower than mean SGCRNA runtime on the identical GSE114007 input.

## Related tools

- **WGCNA** (R package performing the module detection: soft-thresholding power=20, module merge cutHeight=0.9, timed including network/plot generation over 10 runs)
- **SGCRNAs.jl** (Julia package providing the SGCRNA comparison baseline (CGM preprocessing with mode=:FTEST, SpectralClustering, SetNetwork, DrawNetwork) timed on the same GSE114007 matrix) — https://github.com/C37H41N2O6/SGCRNA
- **Julia 1.11** (Runtime environment for the SGCRNA side of the timing comparison)

## Examples

```
Rscript -e 'library(WGCNA); net <- blockwiseModules(datExpr, power=20, mergeCutHeight=0.9)'
```

## Evaluation signals

- timing_comparison.csv contains mean and standard deviation computed over exactly 10 runs per method
- Mean WGCNA runtime across the 10 runs is markedly greater—not equal within noise—than the mean SGCRNA runtime is violated: validation requires WGCNA's mean runtime to be substantially SMALLER than SGCRNA's on the identical GSE114007 input
- Each WGCNA run uses soft-thresholding power=20 and module merge cutHeight=0.9 on the same expression matrix used for SGCRNA
- Network and plot-generation time is included in each of the 10 WGCNA timing measurements
- Both methods consume the identical normalizedCounts_coding.tsv matrix (variables as rows, samples as columns)

## Limitations

- WGCNA relies on the assumption of a scale-free topology, which recent studies suggest is less prevalent than previously assumed
- WGCNA requires complex manual parameter tuning, with the second parameter in particular requiring considerable effort to optimise
- WGCNA disregards regression coefficients: a gene with minimal expression variation but high correlation coefficient may falsely imply co-expression

## Evidence

- [methods] we use the dataset GSE114007, as described in the SGCRNA publication.: [evidence span withheld — anchor status: not_found]
- [intro] WGCNA requires complex manual parameter tuning, with the second parameter in particular requiring considerable effort to optimise: [evidence span withheld — anchor status: not_found]
- [readme] Data = CSV.read("test_data/normalizedCounts_coding.tsv", header=1, comment="#", delim='\t', DataFrame);: [evidence span withheld — anchor status: not_found]
