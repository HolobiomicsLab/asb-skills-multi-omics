---
name: r-implementation-of-mathematical-programming-models
description: Use when an R workflow must select integer decision variables (e.g.,
  per-omic replicate counts) under linear constraints such as minimum power per technology,
  minimum average power, FDR, and a max.size bound.
when_to_use_negative:
- The goal is sample size for machine learning classification — MultiPower's significance-threshold
  and effect-size parameters are not applicable there; use MultiML instead
- Data are not preprocessed and batch-effect free, since MultiPower's parameter estimates
  require cleaned pilot data or manual parameters
- No pilot data and no manually specified parameters are available, so the power curves
  feeding the MILP objective and constraints cannot be estimated
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3517
tools:
- name: MultiPower
  role: Provides the optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize,
    max.size, null.effect) function whose MILP formulation the R implementation reproduces,
    and the reference solution for cross-checking
  repo: github.com/ConesaLab/MultiPower
- name: lpSolve
  role: R MILP solver used to solve the linear integer program with binary assignment
    variables z_n^i; a required MultiPower dependency
  repo: github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:734b771d19234b71cf1571c2f9abde241e7ddc3fc64587a0b1f4bf27e44071ad
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiPower
  - lpSolve
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# r-implementation-of-mathematical-programming-models

## Summary

Implements constrained optimization as a mixed-integer linear program in R using lpSolve, as done in MultiPower's optimalRep function to minimize cost of a multi-omics sample-size design subject to per-omic and average power constraints.

## When to use

Use when an R workflow must select integer decision variables (e.g., per-omic replicate counts) under linear constraints such as minimum power per technology, minimum average power, FDR, and a max.size bound.

## When NOT to use

- The goal is sample size for machine learning classification — MultiPower's significance-threshold and effect-size parameters are not applicable there; use MultiML instead
- Data are not preprocessed and batch-effect free, since MultiPower's parameter estimates require cleaned pilot data or manual parameters
- No pilot data and no manually specified parameters are available, so the power curves feeding the MILP objective and constraints cannot be estimated

## Inputs

- Per-omic power curves over candidate sample sizes (omicPower estimates from pilot data or manual parameter settings)
- Per-omic cost vector
- Parameter object holding omicPower, averagePower, fdr, max.size, equalSize, and null.effect settings
- Optional equalSize flag to couple sample sizes across platforms

## Outputs

- Optimal sample size per omic (integer vector)
- Per-omic achieved powers at the optimal sizes
- Minimal total cost of the design
- Per-omic power study (e.g., written to optimal_sample_sizes.csv)

## How to apply

Encode the objective (total cost across omics) as a linear function of binary auxiliary variables z_n^i, where z_n^i = 1 indicates omic i is assigned sample size n, and pass the model to lpSolve's MILP solver. Add linear constraints: exactly one sample size selected per omic, each omic's power at the selected size meeting its omicPower target, the power averaged across omics meeting averagePower at the chosen fdr and target effect size (Cohen's d), and the sample-size bound max.size; add an equal-size coupling constraint when equalSize is set. Solve the integer program, extract the optimal sample size per omic and the minimal total cost, and cross-check against the package's own optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect) reference. Note that lpSolve is a required dependency of MultiPower, installed via install.packages() before loading the package with library().

## Related tools

- **MultiPower** (Provides the optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect) function whose MILP formulation the R implementation reproduces, and the reference solution for cross-checking) — github.com/ConesaLab/MultiPower
- **lpSolve** (R MILP solver used to solve the linear integer program with binary assignment variables z_n^i; a required MultiPower dependency) — github.com/ConesaLab/MultiPower

## Examples

```
install.packages(c("FDRsampsize","lpSolve")); devtools::install_github("ConesaLab/MultiPower"); opt <- optimalRep(parameters, omicPower=0.8, averagePower=0.8, fdr=0.05, cost, equalSize=FALSE, max.size=50)
```

## Evaluation signals

- Script runs successfully and the MILP returns a feasible integer solution
- All power constraints hold: per-omic power ≥ omicPower, average power ≥ averagePower, and n ≤ max.size for every omic
- MILP optimum (sample sizes and total cost) matches the result returned by the package's own optimalRep() on the same parameter object
- Exactly one sample size is selected per omic (each z_n^i vector sums to 1), and the equal-size coupling is respected when equalSize is set
- Per-omic optimal sample sizes, achieved powers, and total cost are written to optimal_sample_sizes.csv

## Limitations

- MultiPower parameter estimates are optimally calculated from pilot data; without pilot data users must set parameters manually
- Input data must be properly preprocessed and batch effects removed; the tool provides no handling for this
- The power/integer-programming approach does not extend to ML classification sample-size questions, where significance threshold and effect size are not applicable (that is MultiML's scope, which itself requires pilot data)

## Evidence

- [methods] optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect): "- `optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect)`"
- [readme] lpSolve is a required dependency installed from R: "Some dependencies are required before running MultiPower that can be installed from R via 
`install.packages()` and loaded with `library()`:"
- [abstract] MultiPower formulates optimal sample size estimation in a multi-omics experiment: "we formulate the MultiPower method to estimate and assess the optimal sample size in a multi-omics experiment"
- [discussion] MultiPower is not directly applicable to ML classification because its basic power parameters do not apply: "The MultiPower approach is not directly applicable to ML methods used for sample classification, as in this case the basic parameters of the power calculation—significance threshold and effect"
- [discussion] MultiPower parameter estimates come from pilot data or manual input: "Estimates for MultiPower parameters are optimally calculated from pilot data, although they can also be manually provided."
