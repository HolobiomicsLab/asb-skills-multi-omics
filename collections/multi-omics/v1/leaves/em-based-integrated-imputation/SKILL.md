---
name: em-based-integrated-imputation
description: Use when omics data (Z) in a LUCID analysis contain list-wise or sporadic,
  missing-completely-at-random entries that make complete-case analysis infeasible,
  and cluster-specific omics profiles must be estimated jointly with exposures (G)
  and outcome (Y).
when_to_use_negative:
- The omics matrix Z is fully observed — fit the standard LUCID model without the
  missing-data machinery.
- Missingness is not confined to omics data or is not missing completely at random
  — the article's sporadic pattern assumes MCAR entries in Z.
- You need standalone omics imputation with no latent-clustering model (no exposures
  G or outcome Y) — a general-purpose imputer may be more appropriate than imputation
  tied to the LUCID EM fit.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: LUCIDus
  role: R package that fits the LUCID model on G, Z, Y; missing omics values are imputed
    within the EM algorithm (lucid()/estimate_lucid()) instead of as pre-processing
  repo: https://github.com/USCbiostats/LUCIDus
- name: LUCIDus3
  role: Current CRAN release (3.1.0) providing missing-data diagnostics and imputation
    helpers (analyze_missing_pattern(), check_na(), safe_impute(), fill_data(), check_imputation_quality())
    around the EM fit
  repo: https://github.com/USCbiostats/LUCIDus3
- name: mclust
  role: Default initialization of LUCID parameters (mclust plus regression) before
    the EM iterations
- name: R
  role: Implementation and runtime environment for the LUCIDus package and the simulation
    studies
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:1b604bfe039b41f57a86b30b018ec6b9e0f872b5c6e275bb18aa85b92b97a7e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - LUCIDus
  - LUCIDus3
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# em-based-integrated-imputation

## Summary

The LUCID extension (Zhao et al., Bioinformatics Advances 2024, DOI 10.1093/bioadv/vbae123) that imputes missing omics values inside the Expectation-Maximization algorithm used to fit the LUCID latent-cluster model. It handles both list-wise and sporadic (missing-completely-at-random) missing patterns in the omics matrix Z and produces consistent, less biased estimates than mean imputation.

## When to use

Use when omics data (Z) in a LUCID analysis contain list-wise or sporadic, missing-completely-at-random entries that make complete-case analysis infeasible, and cluster-specific omics profiles must be estimated jointly with exposures (G) and outcome (Y).

## When NOT to use

- The omics matrix Z is fully observed — fit the standard LUCID model without the missing-data machinery.
- Missingness is not confined to omics data or is not missing completely at random — the article's sporadic pattern assumes MCAR entries in Z.
- You need standalone omics imputation with no latent-clustering model (no exposures G or outcome Y) — a general-purpose imputer may be more appropriate than imputation tied to the LUCID EM fit.

## Inputs

- Exposure matrix G (individuals x exposures)
- Omics data matrix Z containing list-wise or sporadic (missing-completely-at-random) missing entries
- Outcome vector Y (normal or binary family), optional covariates CoG/CoY
- Model specification: number of latent clusters K (e.g., K = 2), family, seed
- For benchmarking: simulated multi-omics data with known latent clusters and induced sporadic missingness

## Outputs

- Fitted LUCID model object with cluster-specific omics profile parameters and latent cluster assignments
- Within-EM imputed omics values (imputation integrated into the fit, not a separate pre-processing output)
- Bias/consistency comparison of integrated imputation vs impute-mean across replicates (e.g., sporadic_imputation_comparison.csv)

## How to apply

Load LUCIDus in R and assemble the exposure matrix G, omics matrix Z (with missing entries), and outcome Y, then fit with lucid() — or estimate_lucid() in LUCIDus 3.1.0 with lucid_model = 'early', family = 'normal' or 'binary', and a chosen K — so that missing omics values are imputed within the EM iterations rather than as a pre-processing step. Initialize parameters with the default mclust-and-regression scheme before the EM iterations. For the sporadic pattern (entries missing completely at random in individual omics values), do not drop incomplete cases, since complete-case analysis is infeasible in that setting; the integrated imputation uses cluster-specific omics distributions during the E-step. To benchmark, apply mean imputation (impute-mean) to the sporadically missing entries, refit lucid() on the completed matrix, and compute the bias of estimated omics profile parameters against the true values across replicates. In LUCIDus3, diagnose the missing pattern first with analyze_missing_pattern()/check_na() and assess imputation quality afterwards with check_imputation_quality().

## Related tools

- **LUCIDus** (R package that fits the LUCID model on G, Z, Y; missing omics values are imputed within the EM algorithm (lucid()/estimate_lucid()) instead of as pre-processing) — https://github.com/USCbiostats/LUCIDus
- **LUCIDus3** (Current CRAN release (3.1.0) providing missing-data diagnostics and imputation helpers (analyze_missing_pattern(), check_na(), safe_impute(), fill_data(), check_imputation_quality()) around the EM fit) — https://github.com/USCbiostats/LUCIDus3
- **mclust** (Default initialization of LUCID parameters (mclust plus regression) before the EM iterations)
- **R** (Implementation and runtime environment for the LUCIDus package and the simulation studies)

## Examples

```
fit_early <- estimate_lucid(G = G, Z = Z, Y = Y, lucid_model = "early", family = "normal", K = 2, seed = 1008)
```

## Evaluation signals

- Across simulation replicates with sporadic MCAR missingness in Z, integrated imputation yields smaller bias in estimated omics profile parameters than impute-mean + lucid(), reproducing the Supplementary section A pattern.
- Estimates from integrated imputation are consistent: they closely reflect the true underlying simulated omics profiles rather than shrinking toward the global mean.
- The EM fit converges and imputed entries fall within the observed distribution of each omics variable; LUCIDus3's check_imputation_quality() shows acceptable imputation quality.
- Recovered latent cluster structure matches the simulated clusters (e.g., K = 2) instead of being distorted by missing entries.
- Results are tabulated per method (integrated imputation vs impute-mean) and written, e.g., to sporadic_imputation_comparison.csv.

## Limitations

- Sporadic missingness is handled 'when necessary' — the extension was designed primarily for list-wise missingness, and the sporadic pattern assumes missingness completely at random.
- Benchmarks in this context compare only against simple mean imputation (impute-mean); no other imputation methods are evaluated.
- The extracted discussion contains no changelog or documented limitations ('No changelog found.'), so failure modes beyond the list-wise/sporadic MCAR settings are not characterized.

## Evidence

- [abstract] Integrated imputation produces consistent, less biased estimates reflecting true values: "Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values."
- [abstract] Framework targets list-wise missingness and also supports sporadic missingness: "we extend the model to incorporate list-wise missingness within an integrated imputation framework, which can also handle sporadic missingness when necessary."
- [intro] Imputation is integrated within the EM algorithm used for MLE of LUCID: "we use the Expectation-Maximization (EM) algorithm to obtain the maximum likelihood estimates (MLE) of LUCID."
- [intro] Two missing patterns are considered: list-wise and sporadic: "We consider 2 missing patterns in omics data: (1) list-wise missing pattern that only a subset of observations have measured omics data"
- [intro] lucid() fits the LUCID model from exposure matrix G, omics data Z, and outcome Y: "The function `lucid()` is the primary function in the package, which fits a LUCID model based on an exposure matrix (argument `G`), omics data (argument `Z`), outcome data (argument `Y`)"
- [intro] Parameter initialization uses mclust and regression before EM: "The first method (also the default method) uses `mclust` and regression to initialize parameters."
- [readme] LUCIDus3 exposes missing-data diagnostics and imputation for incomplete omics matrices: [evidence span withheld — anchor status: external_doc]
