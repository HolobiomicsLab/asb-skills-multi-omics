---
name: simulation-estimation-bias-assessment
description: Use when you must quantify estimation bias for LUCID (exposures G, omics
  Z, outcome Y) under sporadic, completely-at-random missing omics entries — e.g.,
  complete-case analysis is infeasible and you need to compare integrated imputation
  vs. mean imputation against known ground truth.
when_to_use_negative:
- Missingness is list-wise (whole rows/observations lack omics data) rather than sporadic
  — the paper treats these as two distinct patterns with separate handling
- Real data without known ground truth is the target — bias against true values cannot
  be computed without a simulation with known parameters
- Complete-case analysis is feasible (no sporadic missing omics entries), making imputation
  comparisons unnecessary
tools:
- name: LUCIDus
  role: Fits the LUCID model via lucid(); integrated imputation of missing omics values
    is performed inside the EM algorithm
  repo: https://github.com/USCbiostats/LUCIDus
- name: LUCIDus3
  role: Current CRAN version (3.1.0) adding missing-data diagnostics and imputation
    helpers (analyze_missing_pattern(), check_na(), safe_impute(), check_imputation_quality())
  repo: https://github.com/USCbiostats/LUCIDus3
- name: mclust
  role: Default method used with regression to initialize LUCID parameters in simulation
    fits
- name: R
  role: Implementation language for the simulation, both imputation strategies, and
    bias computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:5866d53efb7217e894529d459cccb3824a4526c7b5ac30397b3e4af473e86b8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - LUCIDus
  - LUCIDus3
  - R
  - mclust
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# simulation-estimation-bias-assessment

## Summary

A simulation-based skill for assessing the bias and consistency of LUCID parameter estimates under sporadic missingness in omics data, benchmarking the integrated imputation framework (imputation inside the EM algorithm) against the simpler impute-mean baseline.

## When to use

Use when you must quantify estimation bias for LUCID (exposures G, omics Z, outcome Y) under sporadic, completely-at-random missing omics entries — e.g., complete-case analysis is infeasible and you need to compare integrated imputation vs. mean imputation against known ground truth.

## When NOT to use

- Missingness is list-wise (whole rows/observations lack omics data) rather than sporadic — the paper treats these as two distinct patterns with separate handling
- Real data without known ground truth is the target — bias against true values cannot be computed without a simulation with known parameters
- Complete-case analysis is feasible (no sporadic missing omics entries), making imputation comparisons unnecessary

## Inputs

- Simulated LUCID data: exposure matrix G, omics matrix Z, outcome Y with known latent clusters
- Missingness configuration (sporadic, missing completely at random in individual omics entries)
- Replication configuration for the simulation (number of replicates, true parameter values)

## Outputs

- Per-replicate bias of estimated omics profile parameters for integrated imputation vs. impute-mean
- Tabulated comparison CSV (e.g., sporadic_imputation_comparison.csv) supporting the bias/consistency conclusion

## How to apply

Simulate multi-omics data with known latent clusters (LUCID structure: exposure matrix G, omics matrix Z, outcome Y), then induce sporadic missingness completely at random in individual Z entries. Fit the LUCID model with lucid() (LUCIDus package), which imputes missing omics values within the EM algorithm. As the competing method, mean-impute the missing Z entries before fitting the same LUCID model. Across replicates, compare each method's estimated omics profile parameters (cluster-specific Z means/covariances) to the true underlying values and compute bias. Rationale: the paper's simulations show the integrated imputation approach yields consistent, less biased estimates that closely reflect true values, whereas mean imputation does not. Tabulate bias for both methods (e.g., sporadic_imputation_comparison.csv) and validate that the pattern matches the published supplementary result.

## Related tools

- **LUCIDus** (Fits the LUCID model via lucid(); integrated imputation of missing omics values is performed inside the EM algorithm) — https://github.com/USCbiostats/LUCIDus
- **LUCIDus3** (Current CRAN version (3.1.0) adding missing-data diagnostics and imputation helpers (analyze_missing_pattern(), check_na(), safe_impute(), check_imputation_quality())) — https://github.com/USCbiostats/LUCIDus3
- **mclust** (Default method used with regression to initialize LUCID parameters in simulation fits)
- **R** (Implementation language for the simulation, both imputation strategies, and bias computation)

## Examples

```
library(LUCIDus); fit <- estimate_lucid(G=G, Z=Z, Y=Y, lucid_model="early", family="normal", K=2, seed=1008)
```

## Evaluation signals

- Integrated imputation produces less biased, more consistent omics profile estimates than impute-mean across replicates, matching Supplementary section A
- Missingness induced is sporadic (individual Z entries, missing completely at random), not list-wise
- Both methods are compared on identical simulated data with identical latent cluster structure (same true parameter values)
- Bias is computed against the known true omics profile parameters per cluster, not just cluster assignment agreement
- Imputation quality can be cross-checked with LUCIDus3 helpers (check_imputation_quality(), analyze_missing_pattern()) before fitting

## Limitations

- Complete-case analysis is infeasible under sporadic missingness, so it cannot serve as a gold-standard comparator in this setting
- Mean imputation ignores the latent cluster structure, which is why it is expected to be more biased — the comparison is specifically for sporadic (MCAR-entry) patterns
- The paper's discussion section provided no extractable limitations or reproducibility gaps ('No changelog found.'), so failure-mode caveats beyond the missingness patterns are not documented

## Evidence

- [abstract] Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values.: "Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values."
- [abstract] Sporadic missingness extension: "we extend the model to incorporate list-wise missingness within an integrated imputation framework, which can also handle sporadic missingness when necessary."
- [intro] Two missing patterns: "We consider 2 missing patterns in omics data: (1) list-wise missing pattern that only a subset of observations have measured omics data and (2) sporadic missing"
- [readme] LUCIDus3 missing-data helpers: [evidence span withheld — anchor status: external_doc]
