---
name: missing-data-likelihood-derivation
description: Use when fitting a LUCID integrative clustering model (G exposures, Z
  omics, Y outcome) where a subset of observations has list-wise missing omics data,
  and you need to modify EM responsibilities and M-step updates for the missing block.
when_to_use_negative:
- Omics data are fully observed — the standard complete-data EM in LUCIDus applies
  and the likelihood partition adds unnecessary complexity.
- Missingness is sporadic (scattered entry-wise) rather than list-wise — the article
  treats sporadic patterns as a separate case the integrated imputation handles 'when
  necessary', so verify the pattern first with analyze_missing_pattern()/check_na().
- The analysis goal is clustering on Z alone without G and Y — the partitioned likelihood's
  missing-subset component relies on G- and Y-related terms.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: LUCIDus
  role: R package whose EM implementation (E-step responsibilities, M-step maximization)
    is modified for the likelihood partition; provides lucid(), estimate_lucid(),
    and imputation utilities
  repo: https://github.com/USCbiostats/LUCIDus
- name: LUCIDus3
  role: Current CRAN (v3.1.0) extension implementing the missing-data EM for early/parallel/serial
    models, with analyze_missing_pattern(), safe_impute(), and check_imputation_quality()
  repo: https://github.com/USCbiostats/LUCIDus3
- name: R
  role: Implementation language for deriving, coding, and testing the modified E-step
    and Q function
- name: mclust
  role: Used with regression to initialize parameters before EM iterations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:f21017d74ce055503da8c8602fa787fa8cab7b7cb5556e90ec4e06e48fabb056
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - LUCIDus
  - LUCIDus3
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# missing-data-likelihood-derivation

## Summary

Derives and implements the likelihood-partition EM algorithm of the LUCID model for list-wise missing omics data, splitting the E-step responsibilities (Eq. 12) and the expected complete-data log-likelihood (Eq. 13) so that omics parameters are updated only from observed-omics rows. This yields an integrated imputation framework within LUCIDus that produces consistent, less biased estimates than complete-case analysis.

## When to use

Use when fitting a LUCID integrative clustering model (G exposures, Z omics, Y outcome) where a subset of observations has list-wise missing omics data, and you need to modify EM responsibilities and M-step updates for the missing block.

## When NOT to use

- Omics data are fully observed — the standard complete-data EM in LUCIDus applies and the likelihood partition adds unnecessary complexity.
- Missingness is sporadic (scattered entry-wise) rather than list-wise — the article treats sporadic patterns as a separate case the integrated imputation handles 'when necessary', so verify the pattern first with analyze_missing_pattern()/check_na().
- The analysis goal is clustering on Z alone without G and Y — the partitioned likelihood's missing-subset component relies on G- and Y-related terms.

## Inputs

- Exposure matrix G (environmental exposures, e.g. prenatal hexachlorobenzene)
- Omics matrix Z with list-wise missing rows (e.g. proteomics for 1301 HELIX children)
- Outcome vector Y (e.g. childhood body mass index)
- Optional covariates CoY
- Number of latent clusters K (integer or vector for model tuning)

## Outputs

- Posterior responsibilities r_imj for latent cluster membership per Eq. 12
- Partitioned expected complete-data log-likelihood (Q function, Eq. 13)
- Updated cluster-specific omics parameters mu_j and Sigma_j from the observed subset {io}
- Fitted LUCID model with integrated imputation of missing omics values
- Imputed omics profiles per latent cluster

## How to apply

Locate the existing EM implementation (E-step responsibilities and M-step maximization) in the LUCIDus source and derive the modified E-step that computes posterior responsibilities r_imj for each latent cluster per Eq. 12, omitting the omics (Z) conditional terms for rows whose omics block is missing. Implement the partitioned Q function of Eq. 13, splitting the expected complete-data log-likelihood into components contributed by observed-omics rows and rows with missing omics, so the missing-subset likelihood contains only G- and Y-related terms. In the M-step, update the cluster-specific multivariate normal parameters mu_j and Sigma_j restricted to the observed-omics subset {io}, while leaving exposure (G) and outcome (Y) parameter updates unchanged across all rows. Fit via estimate_lucid()/lucid() in LUCIDus (K chosen by BIC), then verify EM convergence and compare integrated-imputation estimates against complete-case estimates on simulated data with list-wise missingness.

## Related tools

- **LUCIDus** (R package whose EM implementation (E-step responsibilities, M-step maximization) is modified for the likelihood partition; provides lucid(), estimate_lucid(), and imputation utilities) — https://github.com/USCbiostats/LUCIDus
- **LUCIDus3** (Current CRAN (v3.1.0) extension implementing the missing-data EM for early/parallel/serial models, with analyze_missing_pattern(), safe_impute(), and check_imputation_quality()) — https://github.com/USCbiostats/LUCIDus3
- **R** (Implementation language for deriving, coding, and testing the modified E-step and Q function)
- **mclust** (Used with regression to initialize parameters before EM iterations)

## Examples

```
fit <- estimate_lucid(G = G, Z = Z_with_NA, Y = Y, lucid_model = "early", family = "normal", K = 2, seed = 1008)
```

## Evaluation signals

- EM iterations converge on the simulated dataset with list-wise missing omics (check_convergence() / stable log-likelihood across iterations).
- Responsibilities r_imj for missing-omics rows match Eq. 12 — they omit Z conditional terms and depend only on G and Y components.
- Estimated mu_j and Sigma_j equal the values obtained by restricting the M-step to the observed-omics subset {io}.
- Integrated imputation estimates are consistent and less biased than complete-case estimates in simulation, closely reflecting true underlying values.
- Imputation quality checks (check_imputation_quality()) show imputed Z values closely reflecting true underlying values on held-out entries.

## Limitations

- The likelihood partition is derived for list-wise missingness; sporadic missing patterns are handled 'when necessary' but the derivation targets whole-row missing Z blocks.
- Discussion/limitations content is absent from the provided document ('No changelog found.'), so additional failure modes are not documented in the source.
- mu_j and Sigma_j for clusters are estimable only from rows with observed omics; severe imbalance between {io} and {im} subsets may destabilize cluster-specific covariance estimation.

## Evidence

- [full_text] responsibilities for missing rows ignore Z, per Eq. 12: [evidence span withheld — anchor status: not_found]
- [full_text] eq12_missing_responsibilities: [evidence span withheld — anchor status: not_found]
- [abstract] integrated_imputation_finding: "Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values."
- [intro] missing_patterns: "We consider 2 missing patterns in omics data: (1) list-wise missing pattern that only a subset of observations have measured omics data and (2) sporadic missing pattern that missingness is completely"
- [intro] em_algorithm: "we use the Expectation-Maximization (EM) algorithm to obtain the maximum likelihood estimates (MLE) of LUCID."
- [readme] imputation_helpers: [evidence span withheld — anchor status: not_found]
