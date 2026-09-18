---
name: integer-linear-programming-formulation
description: Use when an experiment design problem must be solved by cost-minimization
  over discrete per-omic sample sizes subject to per-omic power (omicPower), average
  power (averagePower), FDR, max.size, and optional equalSize constraints in a multi-omics
  power study.
when_to_use_negative:
- The goal is sample size for machine-learning sample classification rather than differential
  features analysis — MultiPower's significance threshold and effect size parameters
  are not applicable there; use MultiML instead
- Input data are raw or batch-effect contaminated — MultiPower requires properly preprocessed,
  batch-effect-free data and offers no handling within the tool
- No pilot data are available and no parameters can be set manually — MultiPower parameter
  estimates are optimally calculated from pilot data
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0091
tools:
- name: MultiPower
  role: Source of the power curves, parameters object (omicPower, averagePower, fdr,
    cost, equalSize, max.size) and the optimalRep() reference implementation the MILP
    solution is validated against
  repo: github.com/ConesaLab/MultiPower
- name: lpSolve
  role: MILP solver that receives the linearized 0-1 z_n^i objective and the power/size
    constraints and returns the optimal per-omic sample sizes
  repo: github.com/ConesaLab/MultiPower
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-16937-8
    title: Harmonization of quality metrics and power calculation in multi-omic studies
  source_content_hash: sha256:43df82bafaf2d94b1539d7eb735bc1622c8fc7673bcaf958e56a5e7ad7dfad3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/MultiPower
  tools:
  - MultiPower
  - lpSolve
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2
n_claims: 5
---

# integer-linear-programming-formulation

## Summary

Formulates MultiPower's per-omic optimal sample size search as a 0-1 linear integer program: binary variables z_n^i assign each omic i a sample size n, the objective minimizes total cost, and constraints enforce per-omic and average power targets. Solved with lpSolve and cross-checked against the package's own optimalRep() reference solution.

## When to use

Use when an experiment design problem must be solved by cost-minimization over discrete per-omic sample sizes subject to per-omic power (omicPower), average power (averagePower), FDR, max.size, and optional equalSize constraints in a multi-omics power study.

## When NOT to use

- The goal is sample size for machine-learning sample classification rather than differential features analysis — MultiPower's significance threshold and effect size parameters are not applicable there; use MultiML instead
- Input data are raw or batch-effect contaminated — MultiPower requires properly preprocessed, batch-effect-free data and offers no handling within the tool
- No pilot data are available and no parameters can be set manually — MultiPower parameter estimates are optimally calculated from pilot data

## Inputs

- Per-omic power curves over candidate sample sizes (from MultiPower on pilot/preprocessed multi-omic data)
- Per-omic cost vector
- MultiPower 'parameters' object with omicPower, averagePower, fdr, max.size, equalSize and null.effect settings
- Target effect size (Cohen's d) and FDR level used to derive the power curves

## Outputs

- Optimal sample size per omic (per-omic replicates)
- Achieved per-omic and average power at the optimum
- Minimal total cost of the design
- optimal_sample_sizes.csv with per-omic optimal sample sizes, achieved powers, and total cost

## How to apply

Load MultiPower's per-omic power curves over candidate sample sizes together with the per-omic cost vector and the 'parameters' object carrying omicPower, averagePower, fdr, max.size and equalSize settings. Encode the cost-minimization objective as a linear function of binary auxiliary variables z_n^i, where z_n^i = 1 indicates omic i is assigned sample size n, and hand the MILP to the lpSolve R package. Add linear constraints: exactly one sample size selected per omic, each omic's power at its selected size meeting its omicPower target, the power averaged across omics meeting averagePower, n bounded by max.size, and an equal-size coupling constraint when equalSize is set. Solve the integer program, extract the optimal per-omic sample sizes and minimal total cost, then cross-check against optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect) run on the same parameter object. Rationale: MultiPower's constrained formulation exists because power diminishes as the number of features increases under multiple-testing corrections, so power targets must be negotiated jointly across technologies rather than per-omic in isolation.

## Related tools

- **MultiPower** (Source of the power curves, parameters object (omicPower, averagePower, fdr, cost, equalSize, max.size) and the optimalRep() reference implementation the MILP solution is validated against) — github.com/ConesaLab/MultiPower
- **lpSolve** (MILP solver that receives the linearized 0-1 z_n^i objective and the power/size constraints and returns the optimal per-omic sample sizes) — github.com/ConesaLab/MultiPower

## Examples

```
devtools::install_github("ConesaLab/MultiPower"); opt <- optimalRep(parameters, omicPower=0.8, averagePower=0.8, fdr=0.05, cost, equalSize=FALSE, max.size=NULL, null.effect)
```

## Evaluation signals

- The MILP solution satisfies all power constraints: per-omic power ≥ omicPower, average power ≥ averagePower, and n ≤ max.size
- The MILP optimum matches the result returned by the package's own optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect) on the same parameter object
- Exactly one sample size is selected per omic (each omic's z-variable assignment is a valid partition of the candidate sizes)
- The script runs successfully and writes optimal_sample_sizes.csv containing per-omic optimal sample sizes, achieved powers, and total cost
- The equal-size coupling constraint yields identical n across all omics when equalSize is set

## Limitations

- MultiPower parameter estimates are optimally calculated from pilot data; without pilot data users must set parameters manually
- Input data must be properly preprocessed and batch effects removed before applying MultiPower — no handling within the tool
- The constrained formulation targets differential features analysis; it is not directly applicable to ML sample-classification sample sizing because significance threshold and effect size do not apply there (use MultiML, which itself requires pilot data)
- Some dependencies (FDRsampsize, lpSolve) must be installed from R via install.packages() before running MultiPower

## Evidence

- [methods] MultiPower computes the optimal number of replicates for each omic under a constrained optimization that imposes a minimum power per technology: [evidence span withheld — anchor status: not_found]
- [methods] optimalRep signature from methods: "optimalRep(parameters, omicPower, averagePower, fdr, cost, equalSize, max.size, null.effect)"
- [readme] lpSolve dependency from README: "Some dependencies are required before running MultiPower that can be installed from R via `install.packages()` and loaded with `library()`:"
- [discussion] Power vs number of features rationale from discussion: "Power diminishes as the number of features increases due to the application of multiple testing corrections to control false positives."
- [discussion] Pilot data / preprocessing caveat from discussion: "Data should have been properly preprocessed and eventual batch effects, removed."
