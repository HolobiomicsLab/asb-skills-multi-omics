---
name: lucid-model-em-extension
description: Use when fitting a LUCID model to exposures (G), omics (Z), and outcome
  (Y) data where a subset of observations has list-wise missing omics blocks, and
  you need the EM algorithm modified via the likelihood partition of Eqs. 12-13.
when_to_use_negative:
- The omics missingness is purely sporadic rather than list-wise — the paper's likelihood
  partition and Eq. 12/13 derivation target the list-wise pattern specifically
- You need a general-purpose imputation or clustering method unrelated to the LUCID
  model structure (G, Z, Y with latent clusters)
- The dataset has complete omics data for all observations — the standard unmodified
  LUCIDus EM algorithm applies
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0091
tools:
- name: LUCIDus
  role: R package whose lucid() EM implementation is the starting point; source is
    modified to add the likelihood-partition E-step and Q function
  repo: https://github.com/USCbiostats/LUCIDus
- name: LUCIDus3
  role: Current CRAN-maintained extension (v3.1.0) of LUCIDus incorporating the incomplete-omics
    extensions in estimate_lucid()
  repo: https://github.com/USCbiostats/LUCIDus3
- name: R
  role: Implementation language for the modified E-step, partitioned Q function, and
    M-step updates
- name: mclust
  role: Used with regression as the default method to initialize parameters before
    EM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:fc1108c62ecc2774bf82a4ba26d5258ea14ba6006e7955e552d4b98797e0b1e4
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# lucid-model-em-extension

## Summary

Reconstructs the EM algorithm of the LUCID latent unknown clustering model to handle list-wise missing omics data by partitioning the likelihood: the modified E-step computes responsibilities r_imj omitting the omics (Z) conditional terms for rows with missing omics (Eq. 12), and the M-step updates mu_j and Sigma_j only from the observed-omics subset {io} (Eq. 13).

## When to use

Use when fitting a LUCID model to exposures (G), omics (Z), and outcome (Y) data where a subset of observations has list-wise missing omics blocks, and you need the EM algorithm modified via the likelihood partition of Eqs. 12-13.

## When NOT to use

- The omics missingness is purely sporadic rather than list-wise — the paper's likelihood partition and Eq. 12/13 derivation target the list-wise pattern specifically
- You need a general-purpose imputation or clustering method unrelated to the LUCID model structure (G, Z, Y with latent clusters)
- The dataset has complete omics data for all observations — the standard unmodified LUCIDus EM algorithm applies

## Inputs

- Exposure matrix G (e.g., maternal hexachlorobenzene and other exposures)
- Omics matrix Z with list-wise missing rows (e.g., proteomics data from 1301 children)
- Outcome vector Y (e.g., childhood body mass index)
- Number of latent clusters K for the LUCID model

## Outputs

- Modified E-step responsibilities r_imj for latent clusters under list-wise missingness (Eq. 12)
- Partitioned expected complete-data log-likelihood Q function (Eq. 13)
- M-step updates of cluster-specific mu_j and Sigma_j from the observed subset {io}
- Fitted LUCID model with integrated imputation of missing omics values

## How to apply

Locate the EM implementation (E-step responsibilities and M-step maximization) in the LUCIDus R package source (github.com/USCbiostats/LUCIDus or LUCIDus3). Derive and code the modified E-step that computes posterior responsibilities r_imj per Eq. 12, omitting the omics (Z) conditional terms for individuals whose omics block is list-wise missing, so missing rows contribute likelihood components related only to G and Y. Implement the partitioned Q function of Eq. 13, splitting the expected complete-data log-likelihood into components from observed-omics rows and rows with missing omics. In the M-step, update the cluster-specific multivariate normal parameters mu_j and Sigma_j restricted to observations with observed omics data {io}, while leaving exposure (G) and outcome (Y) parameter updates unchanged. Validate on a small simulated dataset with list-wise missing omics, checking EM convergence and that integrated imputation estimates are consistent and less biased than complete-case estimates, as the simulation studies demonstrate.

## Related tools

- **LUCIDus** (R package whose lucid() EM implementation is the starting point; source is modified to add the likelihood-partition E-step and Q function) — https://github.com/USCbiostats/LUCIDus
- **LUCIDus3** (Current CRAN-maintained extension (v3.1.0) of LUCIDus incorporating the incomplete-omics extensions in estimate_lucid()) — https://github.com/USCbiostats/LUCIDus3
- **R** (Implementation language for the modified E-step, partitioned Q function, and M-step updates)
- **mclust** (Used with regression as the default method to initialize parameters before EM)

## Examples

```
library(LUCIDus); fit <- estimate_lucid(G = G, Z = Z, Y = Y, lucid_model = "early", family = "normal", K = 2, seed = 1008)
```

## Evaluation signals

- E-step responsibilities for missing-omics rows are computed without the Z conditional terms, matching Eq. 12
- The Q function decomposes into observed-omics and missing-omics components as in Eq. 13
- M-step mu_j and Sigma_j estimates change only when observations in {io} change; G and Y parameter updates are unaffected by Z missingness
- On simulated data with list-wise missing omics, EM converges and integrated imputation estimates are consistent and less biased than complete-case estimates
- The fitted model recovers cluster-specific omics profiles (e.g., two proteomics clusters representing higher and lower body mass index in the HELIX application)

## Limitations

- The likelihood partition derivation targets list-wise missing patterns; sporadic missingness is handled 'when necessary' but requires different treatment
- mu_j and Sigma_j are estimated only from the observed-omics subset {io}, so heavily reduced observed subsets may yield unstable covariance estimates
- Discussion-section content in the extracted article is absent ('No changelog found.'), so documented limitations and reproducibility gaps could not be traced

## Evidence

- [methods] Under the likelihood partition for list-wise missingness, the omics-related parameters (mu_j and Sigma_j) are updated in the M-step only from observations in the observed subset {io}, while the likelihood for the missing subset {im} consists only of components related to G and Y (so responsibilities for missing rows ignore Z, per Eq. 12).: [evidence span withheld — anchor status: not_found]
- [intro] We consider 2 missing patterns in omics data: (1) list-wise missing pattern that only a subset of observations have measured omics data and (2) sporadic missing pattern that missingness is completely: "(1) list-wise missing pattern that only a subset of observations have measured omics data"
- [abstract] Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values.: "our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values"
- [abstract] The source code is available at https://github.com/USCbiostats/LUCIDus.: "The source code is available at https://github.com/USCbiostats/LUCIDus."
- [readme] **LUCIDus** is an R package to implement the LUCID model.: "The **LUCIDus** package implements the statistical method LUCID proposed"
