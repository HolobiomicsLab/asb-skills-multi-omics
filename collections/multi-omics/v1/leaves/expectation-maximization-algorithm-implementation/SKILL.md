---
name: expectation-maximization-algorithm-implementation
description: Use when fitting a LUCID model (exposures G, omics Z, outcome Y) by maximum
  likelihood and a subset of observations has list-wise missing omics blocks, requiring
  a modified E-step/M-step that ignores Z terms for missing rows.
when_to_use_negative:
- The omics data Z are fully observed, so the standard (unpartitioned) EM of the original
  LUCID model suffices
- The analysis is supervised clustering outside the LUCID framework (exposures/omics/outcome
  latent structure), e.g. plain mclust-based clustering without outcome integration
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: LUCIDus
  role: R package whose EM implementation (E-step responsibilities, M-step maximization
    in lucid()/estimate_lucid()) is extended and re-implemented for missing omics
  repo: https://github.com/USCbiostats/LUCIDus
- name: R
  role: Language used to derive, code, and test the modified E-step, partitioned Q
    function, and M-step updates
- name: mclust
  role: Used with regression to initialize EM parameters (default initialization method)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:4fb4d0aab421140f8c3bad41587e54e56fad3c88df5dc1cc912ebf602900839e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - LUCIDus
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# expectation-maximization-algorithm-implementation

## Summary

Implements the EM algorithm for the LUCID integrative clustering model, extending the E-step responsibilities and partitioned expected log-likelihood (Q function) to accommodate list-wise missing omics data under a likelihood-partition scheme, so that cluster-specific omics parameters are updated only from observed-omics rows.

## When to use

Use when fitting a LUCID model (exposures G, omics Z, outcome Y) by maximum likelihood and a subset of observations has list-wise missing omics blocks, requiring a modified E-step/M-step that ignores Z terms for missing rows.

## When NOT to use

- The omics data Z are fully observed, so the standard (unpartitioned) EM of the original LUCID model suffices
- The analysis is supervised clustering outside the LUCID framework (exposures/omics/outcome latent structure), e.g. plain mclust-based clustering without outcome integration

## Inputs

- Exposure matrix G
- Omics data matrix Z with list-wise (and optionally sporadic) missing entries
- Outcome vector Y (normal or binary family)
- Optional covariates CoY/CoG
- Number of latent clusters K

## Outputs

- Posterior cluster responsibilities r_imj per individual and cluster
- Maximum likelihood parameter estimates (exposure coefficients, mu_j, Sigma_j, outcome model parameters)
- Cluster assignments and optional integrated omics imputations
- EM convergence diagnostics

## How to apply

Load the LUCIDus source and locate the EM loop inside the fitting routine (estimate_lucid/lucid). Derive the modified E-step computing posterior responsibilities r_imj per Eq. 12, omitting the omics (Z) conditional terms for individuals whose omics block is list-wise missing, so missing-row responsibilities depend only on G and Y. Implement the partitioned Q function of Eq. 13, splitting the expected complete-data log-likelihood into components contributed by observed-omics rows {io} and by rows with missing omics {im}. In the M-step, update the cluster-specific multivariate normal parameters mu_j and Sigma_j restricted to observations with observed omics data, while leaving exposure (G) and outcome (Y) parameter updates unchanged. Test on a small simulated dataset with list-wise missing omics, checking EM convergence and that integrated imputation estimates are consistent and less biased than complete-case estimates.

## Related tools

- **LUCIDus** (R package whose EM implementation (E-step responsibilities, M-step maximization in lucid()/estimate_lucid()) is extended and re-implemented for missing omics) — https://github.com/USCbiostats/LUCIDus
- **R** (Language used to derive, code, and test the modified E-step, partitioned Q function, and M-step updates)
- **mclust** (Used with regression to initialize EM parameters (default initialization method))

## Examples

```
fit <- estimate_lucid(G = G, Z = Z, Y = Y, lucid_model = "early", family = "normal", K = 2, seed = 1008)
```

## Evaluation signals

- EM objective (partitioned Q of Eq. 13) increases monotonically and reaches convergence on simulated data with list-wise missing omics
- Responsibilities for rows with missing omics ignore the Z conditional terms and are driven only by G and Y (Eq. 12 behavior)
- M-step updates of mu_j and Sigma_j are computed exclusively from the observed subset {io}; exposure and outcome parameter updates remain identical to the complete-data case
- Integrated imputation estimates are consistent and less biased than complete-case estimates in simulation
- Model can be validated downstream via summary_lucid() parameter tables and BIC-based K selection

## Limitations

- The likelihood partition directly targets list-wise missingness (only a subset of observations have measured omics); sporadic missingness requires imputation pre-processing or the separate sporadic handling path
- M-step omics parameters are estimated only from the observed subset, so efficiency degrades when the observed-omics subset {io} is small
- Sensitivity to initialization: mclust+regression initialization is the default; numerical stability helpers (e.g. check_and_stabilize_sigma) may be needed for ill-conditioned Sigma_j

## Evidence

- [methods] Under the likelihood partition for list-wise missingness, the omics-related parameters (mu_j and Sigma_j) are updated in the M-step only from observations in the observed subset {io}: [evidence span withheld — anchor status: not_found]
- [methods] Missing-row responsibilities ignore Z (Eq. 12): [evidence span withheld — anchor status: not_found]
- [intro] EM used for MLE of LUCID: "we use the Expectation-Maximization (EM) algorithm to obtain the maximum likelihood estimates (MLE) of LUCID."
- [intro] mclust-based initialization: "The first method (also the default method) uses `mclust` and regression to initialize parameters."
- [abstract] Simulation finding on imputation estimates: "Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values."
- [intro] Two missing patterns considered: "We consider 2 missing patterns in omics data: (1) list-wise missing pattern that only a subset of observations have measured omics data and (2) sporadic missing pattern that missingness is completely"
- [readme] estimate_lucid is the direct fitting API: [evidence span withheld — anchor status: external_doc]
