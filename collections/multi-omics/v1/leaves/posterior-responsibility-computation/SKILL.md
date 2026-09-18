---
name: posterior-responsibility-computation
description: Use when fitting a LUCID latent unknown clustering model on exposures
  (G), omics data (Z), and outcome (Y) where the EM algorithm must assign each observation
  a posterior probability of belonging to each latent cluster, especially when sporadic
  or list-wise missing omics values must be imputed
when_to_use_negative:
- The data have no missing omics values and you only need final cluster labels without
  understanding or re-deriving the EM internals — call the packaged lucid()/estimate_lucid()
  wrapper instead of reimplementing the E-step.
- The omics matrix has block-wise (list-wise) missingness so severe that a separate
  imputation step (e.g., a standalone imputer before clustering) is more appropriate
  than the integrated EM imputation.
- The clustering task involves multiple omics layers in parallel or serial topologies
  (LUCIDus3 'parallel'/'serial' models), where the single-layer E-step of the original
  LUCID model does not directly apply.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_2269
tools:
- name: LUCIDus
  role: R package implementing the LUCID EM algorithm whose E-step computes posterior
    responsibilities and whose Eq. 17 update imputes missing omics values
  repo: https://github.com/USCbiostats/LUCIDus
- name: mclust
  role: Provides the default initialization (Gaussian mixture starting values) for
    the EM algorithm over which responsibilities are computed
- name: LUCIDus3
  role: Extended maintained version of LUCIDus (3.1.0) that supports multi-omics topologies
    and missing-data imputation helpers such as safe_impute() and check_imputation_quality()
  repo: https://github.com/USCbiostats/LUCIDus3
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:28d3795644d93de5f72d6e9fecb934c2d16835418dd618ce46dbe058378aa62a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - LUCIDus
  - LUCIDus3
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# posterior-responsibility-computation

## Summary

Computes posterior cluster responsibilities (E-step probabilities) for every observation within the LUCID EM algorithm, given current exposure, omics, and outcome parameters, and uses these responsibilities to drive both the M-step maximization and the closed-form imputation of missing omics values (Eq. 17).

## When to use

Use when fitting a LUCID latent unknown clustering model on exposures (G), omics data (Z), and outcome (Y) where the EM algorithm must assign each observation a posterior probability of belonging to each latent cluster, especially when sporadic or list-wise missing omics values must be imputed integratively.

## When NOT to use

- The data have no missing omics values and you only need final cluster labels without understanding or re-deriving the EM internals — call the packaged lucid()/estimate_lucid() wrapper instead of reimplementing the E-step.
- The omics matrix has block-wise (list-wise) missingness so severe that a separate imputation step (e.g., a standalone imputer before clustering) is more appropriate than the integrated EM imputation.
- The clustering task involves multiple omics layers in parallel or serial topologies (LUCIDus3 'parallel'/'serial' models), where the single-layer E-step of the original LUCID model does not directly apply.

## Inputs

- Exposure matrix G (environmental exposures, e.g., prenatal hexachlorobenzene levels)
- Omics data matrix Z possibly containing list-wise and sporadic missing values (e.g., HELIX-based simulated proteomics data)
- Outcome data Y (continuous or binary)
- Optional covariates CoY
- Number of latent clusters K and initial parameters from mclust/regression initialization

## Outputs

- Posterior responsibility matrix (N x K probabilities of cluster membership per observation)
- Imputed omics matrix Z with missing entries replaced by conditional expectations under the Gaussian mixture
- Updated model parameters (cluster means, covariances, exposure effects, outcome coefficients) after M-step maximization
- Converged LUCID model object (fitted via lucid()/estimate_lucid())

## How to apply

During the E-step of the LUCID EM algorithm, compute posterior responsibilities for each observation i and each latent cluster k using the current estimates of exposure effects, Gaussian mixture parameters (cluster means and covariances for omics data), and outcome model parameters; the default initialization uses mclust and regression to set starting values (Algorithm 1). For observations with missing omics entries, responsibilities are computed jointly with the missing data: the closed-form solution of Equation (17) replaces each missing entry Z_ib by its conditional expectation under the current responsibilities and estimated Gaussian mixture parameters, combining observed values in the same row with the current responsibility vector, while observed values Z_ia remain fixed. Iterate the E-step responsibility computation and M-step maximization until the EM convergence criterion is met. In LUCIDus this is embedded in the fitting function (lucid()/estimate_lucid()), so responsibilities are recomputed internally at each iteration rather than supplied externally.

## Related tools

- **LUCIDus** (R package implementing the LUCID EM algorithm whose E-step computes posterior responsibilities and whose Eq. 17 update imputes missing omics values) — https://github.com/USCbiostats/LUCIDus
- **mclust** (Provides the default initialization (Gaussian mixture starting values) for the EM algorithm over which responsibilities are computed)
- **LUCIDus3** (Extended maintained version of LUCIDus (3.1.0) that supports multi-omics topologies and missing-data imputation helpers such as safe_impute() and check_imputation_quality()) — https://github.com/USCbiostats/LUCIDus3

## Examples

```
library(LUCIDus); fit <- lucid(G = G, Z = Z, Y = Y, K = 2, seed = 1008)
```

## Evaluation signals

- Responsibilities for each observation sum to 1 across the K latent clusters at every EM iteration (valid posterior normalization).
- Imputed values of sporadically missing omics entries are consistent and minimally biased relative to true underlying values, as verified in the paper's simulation studies.
- EM iterates the responsibility computation and Eq. 17 imputation until the EM convergence criterion is met (no further improvement in the likelihood).
- Observed omics entries Z_ia remain fixed throughout imputation — only missing entries Z_ib are replaced.
- Estimated cluster profiles (e.g., two proteomics clusters representing higher and lower BMI) are interpretable and stable across EM restarts.

## Limitations

- The integrated imputation framework was designed for list-wise missingness with an extension for sporadic missingness; other missing mechanisms (e.g., informative/non-ignorable missingness) are not addressed.
- Imputation quality depends on correct specification of the Gaussian mixture for omics data within each cluster; misspecified cluster means/covariances will bias imputed values.
- Initialization via mclust/regression can affect the EM local optimum reached; multiple starts or the LUCIDus3 numerical-stability helpers (e.g., check_and_stabilize_sigma()) may be needed for convergence robustness.

## Evidence

- [methods] Sporadically missing omics values are updated inside the EM algorithm via the closed-form solution of Equation (17): at each iteration, missing values in Z are imputed using observed values in the same row together with current responsibilities and estimated Gaussian mixture parameters.: [evidence span withheld — anchor status: not_found]
- [abstract] Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values.: "Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values."
- [intro] we use the Expectation-Maximization (EM) algorithm to obtain the maximum likelihood estimates (MLE) of LUCID.: "we use the Expectation-Maximization (EM) algorithm to obtain the maximum likelihood estimates (MLE) of LUCID."
- [intro] The first method (also the default method) uses `mclust` and regression to initialize parameters.: "The first method (also the default method) uses `mclust` and regression to initialize parameters."
- [intro] We consider 2 missing patterns in omics data: (1) list-wise missing pattern that only a subset of observations have measured omics data and (2) sporadic missing pattern that missingness is completely: "We consider 2 missing patterns in omics data: (1) list-wise missing pattern that only a subset of observations"
