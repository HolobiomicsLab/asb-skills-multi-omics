---
name: omics-missingness-simulation
description: Use when a LUCID analysis involves sporadically missing omics entries
  where complete-case analysis is infeasible, and you need evidence that integrated
  EM-based imputation outperforms mean imputation before trusting downstream cluster
  estimates.
when_to_use_negative:
- Missingness is list-wise (whole rows of omics data absent for a subset of observations)
  rather than sporadic entry-level missingness — the paper treats these as two distinct
  patterns and complete-case analysis may be feasible for list-wise subsets
- Real observational data where true omics profile parameters are unknown — simulation-based
  bias evaluation against ground truth does not apply
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3912
tools:
- name: LUCIDus
  role: Fits the LUCID model with integrated EM-based imputation of sporadically missing
    omics values; mean-imputed data fed through the same lucid() workflow as the competing
    method
  repo: https://github.com/USCbiostats/LUCIDus
- name: LUCIDus3
  role: Current CRAN version (3.1.0) providing missing-data diagnostics (analyze_missing_pattern,
    check_na) and imputation utilities (safe_impute, fill_data) supporting the simulation
    comparison
  repo: https://github.com/USCbiostats/LUCIDus3
- name: mclust
  role: Used as the default initialization method (with regression) for LUCID model
    parameters during fitting
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:8810457d202992e70236c13301f7ed8095e6f48d616fd8d4e83675c260bb6de1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - LUCIDus
  - LUCIDus3
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3912
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# omics-missingness-simulation

## Summary

A simulation-based skill for inducing sporadic (MCAR) missingness in multi-omics matrices and benchmarking the LUCID integrated imputation framework (imputation inside the EM algorithm) against mean imputation (impute-mean), by comparing estimated omics profile parameters to known true values across replicates.

## When to use

Use when a LUCID analysis involves sporadically missing omics entries where complete-case analysis is infeasible, and you need evidence that integrated EM-based imputation outperforms mean imputation before trusting downstream cluster estimates.

## When NOT to use

- Missingness is list-wise (whole rows of omics data absent for a subset of observations) rather than sporadic entry-level missingness — the paper treats these as two distinct patterns and complete-case analysis may be feasible for list-wise subsets
- Real observational data where true omics profile parameters are unknown — simulation-based bias evaluation against ground truth does not apply

## Inputs

- Simulated exposure matrix G with known latent cluster structure
- Simulated omics matrix Z with known cluster-specific profile parameters and induced sporadic (MCAR) missing entries
- Simulated outcome Y (continuous or binary) linked to the latent clusters
- R environment with LUCIDus (and mclust for initialization) installed

## Outputs

- Bias/consistency table comparing integrated imputation vs impute-mean (e.g. sporadic_imputation_comparison.csv)
- Estimated omics profile parameters per method across simulation replicates

## How to apply

Set up a LUCID simulation configuration with a known latent cluster structure using an exposure matrix G, omics matrix Z, and outcome Y (e.g. via the sim_data / simulated_HELIX_data objects in LUCIDus). Simulate the omics matrix with known cluster-specific profiles, then induce sporadic missingness completely at random (MCAR) in individual omics entries, as distinct from the list-wise missing pattern. Fit the LUCID model with lucid() (or estimate_lucid() in LUCIDus 3.x) so missing omics values are imputed within the EM algorithm, and fit a competing model on the same data after applying mean imputation to the missing entries. Across replicates, compare estimated omics profile parameters against the true underlying values and compute bias for each method; tabulate the comparison (e.g. sporadic_imputation_comparison.csv). The expected result, per the paper, is that integrated imputation yields consistent, less biased estimates than impute-mean under sporadic missingness.

## Related tools

- **LUCIDus** (Fits the LUCID model with integrated EM-based imputation of sporadically missing omics values; mean-imputed data fed through the same lucid() workflow as the competing method) — https://github.com/USCbiostats/LUCIDus
- **LUCIDus3** (Current CRAN version (3.1.0) providing missing-data diagnostics (analyze_missing_pattern, check_na) and imputation utilities (safe_impute, fill_data) supporting the simulation comparison) — https://github.com/USCbiostats/LUCIDus3
- **mclust** (Used as the default initialization method (with regression) for LUCID model parameters during fitting)

## Examples

```
library(LUCIDus); fit <- estimate_lucid(G = sim_data$G, Z = sim_data$Z, Y = sim_data$Y_normal, lucid_model = "early", family = "normal", K = 2, seed = 1008)
```

## Evaluation signals

- Integrated imputation produces estimates closer to the true simulated omics profile values (lower bias) than mean imputation across replicates
- Integrated imputation estimates are consistent across simulation replicates, matching the pattern reported in Supplementary section A
- Missingness is verified to be MCAR at the individual omics entry level (sporadic pattern), not list-wise
- Both competing methods are fit on identical simulated data with identical latent cluster structure, isolating the imputation strategy as the only difference
- Comparison table (sporadic_imputation_comparison.csv) is populated with per-replicate bias values for both methods

## Limitations

- Complete-case analysis is infeasible under sporadic missingness, so only imputation-based competitors (e.g. impute-mean) can be benchmarked in this setting
- Discussion/limitations content for the article was not extractable ('No changelog found.'), so edge-case caveats beyond the two missing patterns are not documented
- Serial LUCIDus3 models accept only scalar penalties and tuning of K/penalties is limited for serial topologies, which constrains how the simulation can be extended to multi-stage models

## Evidence

- [abstract] Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values.: "Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values."
- [intro] Two missing patterns: list-wise and sporadic: "We consider 2 missing patterns in omics data: (1) list-wise missing pattern that only a subset of observations have measured omics data and (2) sporadic missing pattern that missingness is completely"
- [intro] lucid() fits the LUCID model from G, Z, Y: "The function `lucid()` is the primary function in the package, which fits a LUCID model based on an exposure matrix (argument `G`), omics data (argument `Z`), outcome data (argument `Y`)"
- [readme] LUCIDus handles missingness in omics data: "**LUCIDus** features variable selection, incorporating missingness in omics data, visualization of LUCID model via Sankey diagram, bootstrap inference"
- [readme] LUCIDus3 provides imputation and missingness diagnostics: [evidence span withheld — anchor status: not_found]
