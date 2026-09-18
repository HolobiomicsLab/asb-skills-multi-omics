---
name: multivariate-normal-parameter-estimation
description: Use when fitting a LUCID integrative clustering model where the omics
  block Z is modeled as cluster-specific multivariate normals and some rows have list-wise
  (or sporadic) missing omics that must not contaminate mu_j and Sigma_j updates.
when_to_use_negative:
- The omics block Z is entirely absent or not part of the model — the likelihood then
  consists only of G and Y components and no mu_j/Sigma_j estimation applies.
- Omics data are on a scale poorly approximated by a cluster-specific multivariate
  normal (e.g. raw counts without transformation).
- Only sporadic single-entry missingness needs simple pre-imputation (e.g. safe_impute()/fill_data())
  rather than the integrated list-wise likelihood-partition approach.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: LUCIDus
  role: R package implementing the LUCID EM algorithm with mu_j/Sigma_j M-step updates
    and missing-omics handling
  repo: https://github.com/USCbiostats/LUCIDus
- name: LUCIDus3
  role: Extended maintained version (3.1.0) providing estimate_lucid() with early/parallel/serial
    models and missing-data diagnostics/imputation
  repo: https://github.com/USCbiostats/LUCIDus3
- name: mclust
  role: Used (with regression) as the default initialization of cluster means/covariances
    before EM iterations
- name: R
  role: Language in which the EM E-step/M-step code is written and executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:028d32c488b2f804870219509aaaca5107cef1051e91b4f1c237884a5ba2182f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - LUCIDus
  - LUCIDus3
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# multivariate-normal-parameter-estimation

## Summary

Estimate cluster-specific multivariate normal parameters (mu_j, Sigma_j) for omics data within the LUCID EM algorithm, including the likelihood-partition variant where these parameters are updated only from rows with observed omics under list-wise missingness.

## When to use

Use when fitting a LUCID integrative clustering model where the omics block Z is modeled as cluster-specific multivariate normals and some rows have list-wise (or sporadic) missing omics that must not contaminate mu_j and Sigma_j updates.

## When NOT to use

- The omics block Z is entirely absent or not part of the model — the likelihood then consists only of G and Y components and no mu_j/Sigma_j estimation applies.
- Omics data are on a scale poorly approximated by a cluster-specific multivariate normal (e.g. raw counts without transformation).
- Only sporadic single-entry missingness needs simple pre-imputation (e.g. safe_impute()/fill_data()) rather than the integrated list-wise likelihood-partition approach.

## Inputs

- Exposure matrix G (argument to lucid()/estimate_lucid())
- Omics data matrix Z with list-wise or sporadic missing entries
- Outcome vector Y (continuous or binary, e.g. 'normal' or 'binary' family)
- Optional covariate matrices CoG/CoY
- Number of latent clusters K

## Outputs

- Cluster-specific multivariate normal means mu_j and covariance matrices Sigma_j estimated from the observed-omics subset
- E-step posterior responsibilities r_ij for latent cluster assignment
- Fitted LUCID model object supporting summary_lucid(), predict_lucid(), and integrated imputation of missing omics values

## How to apply

In the LUCID EM algorithm, compute E-step posterior responsibilities r_ij for each latent cluster; under the likelihood partition for list-wise missingness, responsibilities for rows with missing omics omit the Z conditional density terms (per Eq. 12), using only the G and Y components. Build the partitioned Q function (Eq. 13) that splits the expected complete-data log-likelihood into observed-omics and missing-omics components. In the M-step, update mu_j and Sigma_j from weighted observations restricted to the observed subset {io}, leaving exposure (G) and outcome (Y) parameter updates computed from all rows. Initialize means and covariances with mclust plus regression (the LUCIDus default), and apply regularization (lasso for exposures, graphical lasso for the omics covariance) when variable selection is requested. Validate by checking EM convergence on simulated data with list-wise missing omics and confirming the integrated imputation estimates are consistent and less biased than complete-case estimates.

## Related tools

- **LUCIDus** (R package implementing the LUCID EM algorithm with mu_j/Sigma_j M-step updates and missing-omics handling) — https://github.com/USCbiostats/LUCIDus
- **LUCIDus3** (Extended maintained version (3.1.0) providing estimate_lucid() with early/parallel/serial models and missing-data diagnostics/imputation) — https://github.com/USCbiostats/LUCIDus3
- **mclust** (Used (with regression) as the default initialization of cluster means/covariances before EM iterations)
- **R** (Language in which the EM E-step/M-step code is written and executed)

## Examples

```
fit_early <- estimate_lucid(G = G, Z = Z, Y = Y, lucid_model = "early", family = "normal", K = 2, seed = 1008)
```

## Evaluation signals

- Invariant check: M-step updates of mu_j and Sigma_j use only rows in the observed subset {io}; missing-omics rows contribute nothing to these moments.
- E-step responsibilities for missing-omics rows are computed without the Z density terms (Eq. 12), matching G- and Y-based responsibilities only.
- EM iteration reaches convergence (no further change in log-likelihood/responsibilities) on a small simulated dataset with list-wise missing omics.
- Integrated imputation estimates are consistent and less biased than complete-case estimates against known simulated truth.
- Covariance matrices Sigma_j remain valid (positive definite) across iterations; use numerical-stability helpers such as check_and_stabilize_sigma().

## Limitations

- The likelihood partition targets list-wise missing patterns; sporadic missingness is handled only 'when necessary' via the imputation framework.
- Discussion/limitations section content was not extractable ('No changelog found.'), so reproducibility gaps and caveats beyond those stated are undocumented.
- Numerical instability of covariance estimates across EM iterations is a known risk, motivating the package's safe_solve()/check_and_stabilize_sigma() helpers.

## Evidence

- [methods] EM responsibilities omit Z terms for missing rows: [evidence span withheld — anchor status: not_found]
- [methods] M-step restriction to observed subset: [evidence span withheld — anchor status: not_found]
- [intro] EM algorithm is the estimation engine: "we use the Expectation-Maximization (EM) algorithm to obtain the maximum likelihood estimates (MLE) of LUCID."
- [abstract] Integrated imputation quality finding: "Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values."
- [readme] Missing-data handling and covariance regularization in the package: "**LUCIDus** features variable selection, incorporating missingness in
omics data, visualization of LUCID model via Sankey diagram, bootstrap inference"
