---
name: cost-constrained-sample-size-optimization
description: Use when decision support for selecting the optimal number of replicates
  per omic in a multi-omics experimental design when per-technology power targets,
  an average power threshold, an FDR level and platform costs must be balanced simultaneously.
when_to_use_negative:
- The goal is sample size for machine learning sample classification — MultiPower's
  significance threshold and effect size parameters are not applicable there; use
  MultiML instead
- No pilot data are available and parameters cannot be reasonably set manually — MultiPower
  parameter estimates are optimally calculated from pilot data
- Input data are not preprocessed or still contain batch effects — MultiPower provides
  no handling within the tool
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3520
tools:
- name: MultiPower
  role: R package whose optimalRep / MultiPower functions supply the power curves,
    parameters object and reference optimum for the constrained sample-size optimization
  repo: github.com/ConesaLab/MultiPower
- name: lpSolve
  role: MILP solver used to solve the 0-1 linear integer program over binary assignment
    variables z_n^i
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:0a06fc5a540aa35a3fcda399ef9022f8b3793335dda014659023cd831533bcad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/MultiPower
  tools:
  - MultiPower
  - lpSolve
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cost-constrained-sample-size-optimization

## Summary

Formulates MultiPower's optimal sample-size estimation for a multi-omics experiment as a 0-1 linear integer program: minimize total per-omic cost over binary assignment variables z_n^i while enforcing per-omic power, average power, FDR, effect size and max sample size constraints, solved with lpSolve's MILP solver.

## When to use

Decision support for selecting the optimal number of replicates per omic in a multi-omics experimental design when per-technology power targets, an average power threshold, an FDR level and platform costs must be balanced simultaneously.

## When NOT to use

- The goal is sample size for machine learning sample classification — MultiPower's significance threshold and effect size parameters are not applicable there; use MultiML instead
- No pilot data are available and parameters cannot be reasonably set manually — MultiPower parameter estimates are optimally calculated from pilot data
- Input data are not preprocessed or still contain batch effects — MultiPower provides no handling within the tool

## Inputs

- MultiPower 'parameters' object containing per-omic power curves over candidate sample sizes
- Per-omic cost vector (cost)
- omicPower targets per technology
- averagePower threshold across omics
- FDR level (fdr)
- Target effect size (Cohen's d / null.effect)
- Sample-size bound (max.size) and equalSize flag

## Outputs

- Optimal sample size per omic
- Achieved per-omic power study
- Minimal total experiment cost
- optimal_sample_sizes.csv with per-omic sample sizes, achieved powers and total cost

## How to apply

Load the per-omic power curves over candidate sample sizes and the cost vector into MultiPower's 'parameters' object (omicPower, averagePower, fdr, max.size, equalSize settings). Encode the cost-minimization objective as a linear function of binary variables z_n^i, where z_n^i = 1 means omic i is assigned sample size n, in the lpSolve MILP solver. Add linear constraints: exactly one sample size per omic, per-omic power ≥ omicPower target, average power across omics ≥ averagePower, n ≤ max.size, plus an equal-size coupling constraint when equalSize is set. Solve the integer program with optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect), then extract per-omic optimal sample sizes, achieved powers and minimal total cost, and cross-check against the package's own reference solution.

## Related tools

- **MultiPower** (R package whose optimalRep / MultiPower functions supply the power curves, parameters object and reference optimum for the constrained sample-size optimization) — github.com/ConesaLab/MultiPower
- **lpSolve** (MILP solver used to solve the 0-1 linear integer program over binary assignment variables z_n^i)

## Examples

```
devtools::install_github(“ConesaLab/MultiPower”); opt <- optimalRep(parameters, omicPower=0.8, averagePower=0.8, fdr=0.05, cost, equalSize=FALSE, max.size=50, null.effect=0.5)
```

## Evaluation signals

- The MILP solution matches the result returned by MultiPower's own optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect) on the same parameter object
- Exactly one sample size is selected per omic (each omic's assignment variables sum to 1)
- All power constraints hold: per-omic power ≥ omicPower and average power across omics ≥ averagePower at the selected sizes
- Selected sample sizes respect the max.size bound (and equal sample size across platforms when equalSize is set)
- Total cost at the optimum is minimal among all feasible assignments; the script writes per-omic sample sizes, achieved powers and total cost to optimal_sample_sizes.csv

## Limitations

- MultiPower is not directly applicable to ML methods used for sample classification, as the basic power parameters—significance threshold and effect size—are not applicable in that setting
- Estimates for MultiPower parameters are optimally calculated from pilot data, although they can also be manually provided; without pilot data the user must set them by hand
- Data should be properly preprocessed and batch effects removed before applying MultiPower; no handling is provided within the tool
- Power diminishes as the number of features increases due to multiple testing corrections controlling false positives

## Evidence

- [abstract] we formulate the MultiPower method to estimate and assess the optimal sample size in a multi-omics experiment: "we formulate the MultiPower method to estimate and assess the optimal sample size in a multi-omics experiment"
- [methods] optimalRep signature: "- `optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect)`"
- [discussion] MultiPower parameter dependence on pilot data: "Estimates for MultiPower parameters are optimally calculated from pilot data, although they can also be manually provided."
- [readme] lpSolve dependency: "Some dependencies are required before running MultiPower that can be installed from R via 
`install.packages()` and loaded with `library()`:

- FDRsampsize
- lpSolve"
