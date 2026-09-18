---
name: constraint-optimization-validation-against-reference
description: Use when designing a multi-omics experiment that needs an optimal number
  of replicates per omic under minimum power, average power, FDR, effect size and
  cost constraints, and the optimization result must be cross-checked against the
  package reference.
when_to_use_negative:
- The goal is sample size for machine-learning sample classification — MultiPower's
  significance threshold and effect size parameters are not applicable there; use
  MultiML instead
- The input data are raw or contain batch effects — MultiPower requires properly preprocessed,
  batch-effect-free pilot data and performs no preprocessing itself
- No pilot data exist and parameters cannot be set manually — MultiPower parameter
  estimates are optimally calculated from pilot data
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: MultiPower
  role: Provides the power-study parameters object, the optimalRep reference implementation,
    and the per-omic power curves the MILP must reproduce
  repo: github.com/ConesaLab/MultiPower
- name: lpSolve
  role: MILP solver used to solve the binary-variable cost-minimization integer program
    with the power and sample-size constraints
  repo: github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:b202cd36b8f50ea6b2628eb32a15c39ef86a5fae0192c03ca6e544a8256b8b51
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/MultiPower
  tools:
  - MultiPower
  - lpSolve
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# constraint-optimization-validation-against-reference

## Summary

Encodes MultiPower's per-omic sample-size selection as a mixed-integer linear program solved with lpSolve, imposing per-omic power (omicPower), average power (averagePower), FDR, effect size (Cohen's d), cost, max.size, and optional equalSize constraints, then validates the MILP optimum against the package's own optimalRep() reference solution.

## When to use

Use when designing a multi-omics experiment that needs an optimal number of replicates per omic under minimum power, average power, FDR, effect size and cost constraints, and the optimization result must be cross-checked against the package reference.

## When NOT to use

- The goal is sample size for machine-learning sample classification — MultiPower's significance threshold and effect size parameters are not applicable there; use MultiML instead
- The input data are raw or contain batch effects — MultiPower requires properly preprocessed, batch-effect-free pilot data and performs no preprocessing itself
- No pilot data exist and parameters cannot be set manually — MultiPower parameter estimates are optimally calculated from pilot data

## Inputs

- Per-omic power curves over candidate sample sizes (MultiPower 'parameters' object with omicPower, averagePower, fdr, max.size, equalSize settings)
- Per-omic cost vector
- Target effect size (Cohen's d) / null.effect value
- FDR level (fdr) and minimum average power (averagePower) thresholds
- Optional equalSize flag and max.size sample-size bound

## Outputs

- Per-omic optimal sample sizes (written to optimal_sample_sizes.csv)
- Achieved per-omic and average power at the selected sample sizes
- Minimal total cost of the optimal design
- MILP-vs-optimalRep comparison confirming the optimum matches the reference solution

## How to apply

Load the per-omic power curves over candidate sample sizes and the per-omic cost vector into MultiPower's 'parameters' object (omicPower, averagePower, fdr, max.size, equalSize settings). Reformulate the cost-minimization objective as a linear function of binary auxiliary variables z_n^i (z_n^i = 1 when omic i is assigned sample size n) and solve with the lpSolve MILP solver, which must be installed as a MultiPower dependency. Add the linear constraints: exactly one sample size per omic, per-omic power at the selected size meeting its omicPower target, the power averaged across omics meeting averagePower, and the max.size bound (with an equal-size coupling constraint when equalSize is set). Solve the integer program, extract per-omic optimal sample sizes and minimal total cost, then cross-check against optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect) on the same parameter object. Rationale: power falls as the number of features grows due to multiple-testing correction, so constraints must be set against the FDR-adjusted per-omic power curves rather than raw power.

## Related tools

- **MultiPower** (Provides the power-study parameters object, the optimalRep reference implementation, and the per-omic power curves the MILP must reproduce) — github.com/ConesaLab/MultiPower
- **lpSolve** (MILP solver used to solve the binary-variable cost-minimization integer program with the power and sample-size constraints) — github.com/ConesaLab/MultiPower

## Examples

```
optimalRep(parameters, omicPower=0.8, averagePower=0.8, fdr=0.05, cost=cost, equalSize=FALSE, max.size=50, null.effect=0.5)
```

## Evaluation signals

- The script runs successfully and the lpSolve MILP reaches an optimal solution with all constraints satisfied
- Per-omic power at the selected sample size is ≥ omicPower for every omic
- Power averaged across omics is ≥ averagePower, and every selected n ≤ max.size
- The MILP optimum matches the result returned by optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect) on the same parameter object
- Exactly one sample size is selected per omic (each z-vector sums to 1), and per-omic sizes, achieved powers, and total cost are written to optimal_sample_sizes.csv

## Limitations

- MultiPower parameter estimates are optimally calculated from pilot data, although they can also be manually provided
- Input data must be properly preprocessed and batch effects removed before applying MultiPower; the tool does not handle this
- lpSolve (and FDRsampsize) are required dependencies that must be installed before running MultiPower

## Evidence

- [methods] optimalRep reference signature: "optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect)"
- [readme] lpSolve dependency in README: "Some dependencies are required before running MultiPower that can be installed from R via `install.packages()` and loaded with `library()`"
- [abstract] MultiPower purpose from abstract: "we formulate the MultiPower method to estimate and assess the optimal sample size in a multi-omics experiment"
- [discussion] power decreases with feature count: "Power diminishes as the number of features increases due to the application of multiple testing corrections to control false positives."
