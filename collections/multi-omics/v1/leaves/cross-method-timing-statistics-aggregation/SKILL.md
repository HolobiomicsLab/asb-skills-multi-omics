---
name: cross-method-timing-statistics-aggregation
description: Use when you need to quantify and compare the computational cost of two
  analysis pipelines (e.g., SGCRNA's spectral-clustering eigenvector computation vs
  WGCNA) on an identical transcriptomic dataset, with replicated timed runs and statistical
  aggregation.
when_to_use_negative:
- When the goal is to compare biological results (module membership or hub genes)
  rather than execution time — use the methods' own output-comparison workflows instead
- When the two methods are run on different inputs or different subsets of the GSE114007
  matrix — timing differences would be confounded and the comparison invalid
- When a single run per method suffices for a rough estimate but you need publication-grade
  mean±sd statistics — this skill requires 10 replicated runs per method
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SGCRNAs.jl
  role: Julia implementation of SGCRNA whose full pipeline (CGM :FTEST, SpectralClustering,
    SetNetwork, DrawNetwork) is timed in 10 replicated runs
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: Julia 1.11
  role: Runtime environment in which SGCRNA runs are executed and wall-clock timed
- name: WGCNA
  role: R baseline method timed in 10 replicated runs on the same expression matrix
    with power=20 and cutHeight=0.9, including plot generation
- name: CSV.jl / DataFrames.jl
  role: Julia packages used to load normalizedCounts_coding.tsv with variables as
    rows and samples as columns before timing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:364d5c2e06cbcbdb46bc45a78c0f5146548e2f2ccb79d9f32dc423fa662fa59d
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
  - CSV.jl / DataFrames.jl
  - Julia 1.11
  - SGCRNAs.jl
  - WGCNA
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.14285714285714285
n_claims: 7
---

# cross-method-timing-statistics-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill for comparing wall-clock execution time of two co-expression network analysis methods (SGCRNA in Julia vs WGCNA in R) on the same input matrix, aggregating repeated runs into mean and standard deviation timing statistics.

## When to use

Use when you need to quantify and compare the computational cost of two analysis pipelines (e.g., SGCRNA's spectral-clustering eigenvector computation vs WGCNA) on an identical transcriptomic dataset, with replicated timed runs and statistical aggregation.

## When NOT to use

- When the goal is to compare biological results (module membership or hub genes) rather than execution time — use the methods' own output-comparison workflows instead
- When the two methods are run on different inputs or different subsets of the GSE114007 matrix — timing differences would be confounded and the comparison invalid
- When a single run per method suffices for a rough estimate but you need publication-grade mean±sd statistics — this skill requires 10 replicated runs per method

## Inputs

- GSE114007 RNA expression matrix normalizedCounts_coding.tsv (variables as rows, samples as columns)
- SraRunTable.csv clinical/phenotype data (used in the SGCRNA pipeline steps being timed)
- SGCRNAs.jl Julia package (Julia 1.11)
- R WGCNA package with soft-thresholding power=20 and merge cutHeight=0.9

## Outputs

- timing_comparison.csv with mean and standard deviation of 10 runtimes per method
- Per-run wall-clock time measurements for SGCRNA (CGM + SpectralClustering + SetNetwork + DrawNetwork)
- Per-run wall-clock time measurements for WGCNA including network/plot generation

## How to apply

Load the identical expression matrix (GSE114007 normalizedCounts_coding.tsv, variables as rows and samples as columns) into both pipelines so the comparison is on identical input. In Julia 1.11 with the SGCRNAs.jl package, time 10 full runs of the complete pipeline — CGM preprocessing with mode=:FTEST, SpectralClustering (tNodeNum=100, depthMax=5 defaults), SetNetwork, and DrawNetwork — recording wall-clock time per run. In R, time 10 WGCNA runs on the same matrix using soft-thresholding power=20 and module merge cutHeight=0.9, including network and plot-generation time in each measurement so the scope matches the SGCRNA runs. Aggregate the 10 runtimes per method into mean and standard deviation and write the summary to a timing table (timing_comparison.csv). Judge correctness by checking that the mean SGCRNA runtime is markedly larger than the mean WGCNA runtime — the difference must exceed run-to-run noise, not be equal within noise.

## Related tools

- **SGCRNAs.jl** (Julia implementation of SGCRNA whose full pipeline (CGM :FTEST, SpectralClustering, SetNetwork, DrawNetwork) is timed in 10 replicated runs) — https://github.com/C37H41N2O6/SGCRNA
- **Julia 1.11** (Runtime environment in which SGCRNA runs are executed and wall-clock timed)
- **WGCNA** (R baseline method timed in 10 replicated runs on the same expression matrix with power=20 and cutHeight=0.9, including plot generation)
- **CSV.jl / DataFrames.jl** (Julia packages used to load normalizedCounts_coding.tsv with variables as rows and samples as columns before timing)

## Examples

```
using CSV, DataFrames, SGCRNAs; Data = CSV.read("normalizedCounts_coding.tsv", header=1, comment="#", delim='\t', DataFrame); @time CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST);
```

## Evaluation signals

- timing_comparison.csv contains exactly 10 replicated runtime measurements per method (SGCRNA and WGCNA) plus aggregated mean and standard deviation per method
- Both methods were run on the identical GSE114007 expression matrix (normalizedCounts_coding.tsv) — same rows and columns in Julia and R
- WGCNA timing includes soft-thresholding power=20, merge cutHeight=0.9, and network/plot-generation time, matching the full SGCRNA pipeline scope (CGM through DrawNetwork)
- Validation: mean SGCRNA runtime across the 10 runs is markedly greater (not equal within noise) than the mean WGCNA runtime on the identical input
- SGCRNA runs use CGM mode=:FTEST and SpectralClustering defaults (tNodeNum=100, depthMax=5), confirming the timed configuration matches the paper's workflow

## Limitations

- Timing results are specific to dataset GSE114007 and the chosen SGCRNA parameters (mode=:FTEST, tNodeNum=100, depthMax=5); other datasets or parameter settings may shift the relative runtimes
- SGCRNA's spectral-clustering eigenvector computation is the source of its higher runtime, so comparisons on very small matrices may not show a marked difference
- The WGCNA measurement deliberately includes network and plot-generation time; excluding these would change the comparison scope and invalidate the replication of the paper's result
- No changelog found for the SGCRNAs.jl repository, so version-specific performance behavior cannot be tracked

## Evidence

- [methods] Timing comparison of SGCRNA vs WGCNA: [evidence span withheld — anchor status: not_found]
- [readme] README data loading: [evidence span withheld — anchor status: not_found]
