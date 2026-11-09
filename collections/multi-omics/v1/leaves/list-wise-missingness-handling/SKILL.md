---
name: list-wise-missingness-handling
description: Use when a multi-omics dataset shows list-wise missingness — entire omics
  (Z) rows missing for a subset of individuals — and LUCID integrated clustering of
  G, Z, and Y must still be fit.
when_to_use_negative:
- Omics data are complete for all individuals — the standard unpartitioned LUCID EM
  applies and the likelihood partition adds no value.
- Missingness is sporadic (scattered cells missing completely at random) rather than
  list-wise blocks — the paper treats this as a distinct missing pattern requiring
  different likelihood handling.
- Only G and Y are available (no omics matrix Z) — there is no omics likelihood term
  to partition or impute.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3365
tools:
- name: LUCIDus
  role: R package whose LUCID EM implementation (E-step responsibilities, M-step maximization
    in lucid()) is extended with the likelihood-partition for list-wise missing omics
  repo: https://github.com/USCbiostats/LUCIDus
- name: LUCIDus3
  role: Current CRAN-maintained version (3.1.0) exposing estimate_lucid() with missing-data
    handling during fitting and missingness diagnostics/imputation (analyze_missing_pattern,
    safe_impute, fill_data)
  repo: https://github.com/USCbiostats/LUCIDus3
- name: R
  role: Implementation language for the modified E-step, partitioned Q function, and
    restricted M-step updates
- name: mclust
  role: Default method used with regression to initialize LUCID parameters before
    EM iterations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:b9f855395e5c92e78735769cd8f57504574781e676a6707703d468773babd33e
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
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3365
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# list-wise-missingness-handling

## Summary

Extends the LUCID EM algorithm to list-wise missing omics by partitioning the likelihood: E-step responsibilities for missing rows (r_imj, Eq. 12) drop the omics (Z) conditional terms, and M-step updates of cluster-specific mu_j and Sigma_j use only the observed-omics subset {io}. The resulting integrated imputation yields consistent, less biased estimates than complete-case analysis.

## When to use

Use when a multi-omics dataset shows list-wise missingness — entire omics (Z) rows missing for a subset of individuals — and LUCID integrated clustering of G, Z, and Y must still be fit.

## When NOT to use

- Omics data are complete for all individuals — the standard unpartitioned LUCID EM applies and the likelihood partition adds no value.
- Missingness is sporadic (scattered cells missing completely at random) rather than list-wise blocks — the paper treats this as a distinct missing pattern requiring different likelihood handling.
- Only G and Y are available (no omics matrix Z) — there is no omics likelihood term to partition or impute.

## Inputs

- Exposure matrix G (e.g., maternal exposures such as prenatal hexachlorobenzene from the HELIX study)
- Omics matrix Z with list-wise missing rows (e.g., incomplete proteomics data from 1301 children)
- Outcome vector Y (continuous or binary; family = 'normal' or 'binary' in LUCIDus)
- Number of latent clusters K (selected via BIC in the LUCIDus workflow)

## Outputs

- Posterior cluster responsibilities r_ij (Eq. 12), with the Z conditional terms omitted for missing-omics rows
- Partitioned expected complete-data log-likelihood Q (Eq. 13) split over {io} and {im}
- Cluster-specific omics parameters mu_j and Sigma_j estimated only from the observed-omics subset {io}
- Integrated imputation of the missing omics block within the fitted LUCID model

## How to apply

Locate the EM implementation in LUCIDus (lucid()/estimate_lucid()) and index rows into an observed-omics subset {io} and a missing-omics subset {im} to partition the full-data likelihood. In the modified E-step, compute posterior responsibilities r_imj per Eq. 12 by omitting the multivariate-normal omics terms, so responsibilities for missing rows depend only on G and Y. Implement the partitioned expected complete-data log-likelihood (Eq. 13), splitting Q into components contributed by observed-omics rows and by rows with missing omics. In the M-step, update cluster-specific means mu_j and covariance matrices Sigma_j restricted to observations with observed omics, while exposure (G) and outcome (Y) parameter updates remain based on all rows. Run EM to convergence (with K tuned via BIC in the LUCIDus workflow) so imputation of the missing omics block is integrated into model estimation rather than done as a separate preprocessing step. Validate on a small simulated dataset with induced list-wise missing omics: check EM convergence and confirm the integrated-imputation estimates are consistent and less biased than complete-case estimates.

## Related tools

- **LUCIDus** (R package whose LUCID EM implementation (E-step responsibilities, M-step maximization in lucid()) is extended with the likelihood-partition for list-wise missing omics) — https://github.com/USCbiostats/LUCIDus
- **LUCIDus3** (Current CRAN-maintained version (3.1.0) exposing estimate_lucid() with missing-data handling during fitting and missingness diagnostics/imputation (analyze_missing_pattern, safe_impute, fill_data)) — https://github.com/USCbiostats/LUCIDus3
- **R** (Implementation language for the modified E-step, partitioned Q function, and restricted M-step updates)
- **mclust** (Default method used with regression to initialize LUCID parameters before EM iterations)

## Examples

```
mp <- analyze_missing_pattern(Z); fit <- estimate_lucid(G = G, Z = Z, Y = Y, lucid_model = "early", family = "normal", K = 2, seed = 1008)
```

## Evaluation signals

- EM converges on a small simulated dataset with induced list-wise missing omics, with the integrated-imputation estimates consistent and less biased than complete-case estimates
- Responsibilities r_imj for missing-omics rows are invariant to the (missing) Z values, i.e., they depend only on G and Y as Eq. 12 requires
- After the M-step, mu_j and Sigma_j equal the responsibility-weighted estimates computed over rows in the observed subset {io} only
- Exposure (G) and outcome (Y) parameter updates are unchanged relative to the standard LUCID EM, changing only the omics-block updates
- Applied to the HELIX/Exposome Data Challenge data, the model recovers proteomics profiles for two clusters corresponding to higher and lower childhood BMI

## Limitations

- This likelihood partition targets the list-wise missing pattern only; sporadic missingness is a second pattern with different handling, though the framework can also handle sporadic missingness when necessary.
- Cluster-specific omics parameters (mu_j, Sigma_j) are estimated only from the observed subset {io}, so a small or unrepresentative observed subset can degrade omics-profile estimation.
- Discussion-section limitations were not extractable from the source ('No changelog found.'), so additional numerical or identifiability caveats are undocumented.

## Evidence

- [abstract] A significant challenge in multi-omics analysis is the issue of list-wise missingness.: "A significant challenge in multi-omics analysis is the issue of list-wise missingness."
- [methods] posterior responsibilities r_imj for each latent cluster according to Eq. 12, omitting the omics (Z) conditional terms for individuals whose omics block is list-wise missing: [evidence span withheld — anchor status: not_found]
- [methods] the likelihood for the missing subset {im} consists only of components related to G and Y (so responsibilities for missing rows ignore Z, per Eq. 12): [evidence span withheld — anchor status: not_found]
- [methods] Write the M-step updates of the cluster-specific multivariate normal parameters mu_j and Sigma_j restricted to observations with observed omics data: [evidence span withheld — anchor status: not_found]
- [intro] we use the Expectation-Maximization (EM) algorithm to obtain the maximum likelihood estimates (MLE) of LUCID.: "we use the Expectation-Maximization (EM) algorithm to obtain the maximum likelihood estimates (MLE) of LUCID."
- [abstract] Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values.: "Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values."
- [readme] **Missing-data diagnostics and imputation** for incomplete omics matrices.: [evidence span withheld — anchor status: external_doc]
