---
name: co-expression-network-runtime-benchmarking
description: Use when comparing computational cost of co-expression network methods
  (e.g., SGCRNA vs WGCNA) on an identical expression matrix, where timing differences
  from spectral-clustering eigenvector computation must be quantified across repeated
  runs.
when_to_use_negative:
- When the goal is to assess biological module quality or biomarker discovery rather
  than execution-time comparison of the two methods
- When only a single run per method is feasible — the comparison requires 10 repeated
  runs per method to aggregate mean and standard deviation
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: SGCRNAs
  role: Julia package timed in the benchmark; provides CGM (preprocessing, mode=:FTEST),
    SpectralClustering, SetNetwork, and DrawNetwork steps
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: WGCNA
  role: R-based co-expression method timed as the baseline comparator with soft-thresholding
    power=20 and cutHeight=0.9
- name: Julia 1.11
  role: Runtime environment in which SGCRNA timing runs are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:06057a3bc55cfb35ed725c37da6d7c895469e814fc918ec9600caf2b8c6fe146
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
  - SGCRNAs
  - WGCNA
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.16666666666666666
n_claims: 12
---

# co-expression-network-runtime-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the execution time of SGCRNA (spectral-clustering-guided co-expression analysis in Julia) against WGCNA (R) on the same transcriptomic dataset, as done for GSE114007, where SGCRNA's spectral-clustering eigenvector computation makes it markedly slower than WGCNA.

## When to use

Use when comparing computational cost of co-expression network methods (e.g., SGCRNA vs WGCNA) on an identical expression matrix, where timing differences from spectral-clustering eigenvector computation must be quantified across repeated runs.

## When NOT to use

- When the goal is to assess biological module quality or biomarker discovery rather than execution-time comparison of the two methods
- When only a single run per method is feasible — the comparison requires 10 repeated runs per method to aggregate mean and standard deviation

## Inputs

- normalizedCounts_coding.tsv RNA expression matrix (variables as rows, samples as columns) from GSE114007
- SGCRNAs.jl package installed in Julia 1.11
- WGCNA R package with soft-thresholding power=20 and merge cutHeight=0.9

## Outputs

- timing_comparison.csv with mean and standard deviation of 10 runtimes per method (SGCRNA, WGCNA)
- Per-run wall-clock timing records for SGCRNA and WGCNA including network/plot generation

## How to apply

Load the GSE114007 RNA expression matrix (normalizedCounts_coding.tsv, variables as rows, samples as columns) with CSV.jl/DataFrames.jl in Julia 1.11. Time 10 full SGCRNA runs using the SGCRNAs.jl package, each covering CGM preprocessing in mode=:FTEST, SpectralClustering (tNodeNum=100, depthMax=5 defaults), SetNetwork, and DrawNetwork, recording wall-clock time per run. Time 10 WGCNA runs in R on the same matrix using soft-thresholding power=20 and module merge cutHeight=0.9, including network/plot-generation time in each measurement. Aggregate mean and standard deviation of the 10 runtimes per method into timing_comparison.csv, and validate that mean SGCRNA runtime is substantially (markedly, not within noise) larger than mean WGCNA runtime on the identical input.

## Related tools

- **SGCRNAs** (Julia package timed in the benchmark; provides CGM (preprocessing, mode=:FTEST), SpectralClustering, SetNetwork, and DrawNetwork steps) — https://github.com/C37H41N2O6/SGCRNA
- **WGCNA** (R-based co-expression method timed as the baseline comparator with soft-thresholding power=20 and cutHeight=0.9)
- **Julia 1.11** (Runtime environment in which SGCRNA timing runs are executed)

## Examples

```
using CSV, DataFrames, SGCRNAs; Data = CSV.read("normalizedCounts_coding.tsv", header=1, comment="#", delim='\t', DataFrame); CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST); clust, pos, edge_data = SpectralClustering(CorData, GradData);
```

## Evaluation signals

- timing_comparison.csv contains mean and standard deviation computed over exactly 10 runs per method
- Mean SGCRNA runtime across the 10 runs is markedly greater (not equal within noise) than the mean WGCNA runtime on the identical GSE114007 input
- Each SGCRNA run includes the full pipeline: CGM (mode=:FTEST), SpectralClustering, SetNetwork, and DrawNetwork
- Each WGCNA timing measurement includes network and plot-generation time, not just clustering time
- Both methods are timed on the same normalizedCounts_coding.tsv expression matrix

## Limitations

- SGCRNA requires substantially more execution time than WGCNA because spectral clustering performs eigenvector computation
- WGCNA timing depends on manually determined parameters (soft-thresholding power=20, cutHeight=0.9), which require considerable tuning effort

## Evidence

- [other] SGCRNA requires substantially more execution time than WGCNA; the timing comparison was conducted on dataset GSE114007.: [evidence span withheld — anchor status: not_found]
- [readme] README usage of the SGCRNAs pipeline on GSE114007 data: "CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST);"
- [intro] WGCNA parameter tuning burden: "Two essential parameters that must be manually determined, with the second parameter in particular requiring considerable"
