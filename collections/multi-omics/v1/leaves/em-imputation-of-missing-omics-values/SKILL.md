---
name: em-imputation-of-missing-omics-values
description: Use when an omics matrix (e.g., proteomics from the HELIX cohort) has
  sporadic or list-wise missing entries and you are fitting a LUCID latent-cluster
  model and want missing values imputed within the EM iteration.
when_to_use_negative:
- The omics matrix is complete (no missing values), so imputation machinery is unnecessary
  and `lucid()` can be fit directly
- Missingness is so extensive that a subset of observations has no measured omics
  data at all and a list-wise deletion or two-stage approach is more appropriate than
  within-EM imputation
- The analysis goal is generic imputation without joint clustering — use a standalone
  imputation tool instead, since Eq. 17 imputation is defined only within the LUCID
  EM framework
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: LUCIDus
  role: R package implementing the LUCID model with the integrated EM imputation update
    (Eq. 17) for incomplete omics data
  repo: https://github.com/USCbiostats/LUCIDus
- name: mclust
  role: Default method, together with regression, for initializing LUCID model parameters
    before EM iteration
- name: LUCIDus3
  role: Extended CRAN package (3.1.0) offering missing-data diagnostics and imputation
    helpers (analyze_missing_pattern, check_na, safe_impute, fill_data) for incomplete
    omics matrices
  repo: https://github.com/USCbiostats/LUCIDus3
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:e9f0d4547d664194251c26655cd7d94d2c43d6510a9e519e1f9207522155c641
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - LUCIDus
  - LUCIDus3
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# em-imputation-of-missing-omics-values

## Summary

Imputes sporadically missing omics entries inside the LUCID EM algorithm using the closed-form conditional expectation of Equation (17), so that imputation and cluster estimation are performed jointly rather than as a separate preprocessing step.

## When to use

Use when an omics matrix (e.g., proteomics from the HELIX cohort) has sporadic or list-wise missing entries and you are fitting a LUCID latent-cluster model and want missing values imputed within the EM iteration.

## When NOT to use

- The omics matrix is complete (no missing values), so imputation machinery is unnecessary and `lucid()` can be fit directly
- Missingness is so extensive that a subset of observations has no measured omics data at all and a list-wise deletion or two-stage approach is more appropriate than within-EM imputation
- The analysis goal is generic imputation without joint clustering — use a standalone imputation tool instead, since Eq. 17 imputation is defined only within the LUCID EM framework

## Inputs

- Exposure matrix G (e.g., prenatal hexachlorobenzene levels from the HELIX-based simulated dataset)
- Omics matrix Z with list-wise and sporadic missing values (e.g., proteomics data from 1301 children)
- Outcome data Y (e.g., childhood body mass index)
- Optional covariates CoY

## Outputs

- Completed omics matrix with missing Z entries replaced by conditional-expectation imputations
- Fitted LUCID model with posterior cluster responsibilities and Gaussian mixture parameters (cluster means, covariances)
- Latent cluster assignments linking exposures to omics profiles (e.g., higher- vs lower-BMI proteomics clusters)

## How to apply

Load the omics matrix Z containing list-wise and/or sporadic missing patterns into the LUCIDus package and initialize model parameters with the default mclust-and-regression scheme (Algorithm 1). In the E-step, compute posterior cluster responsibilities for every observation given current exposure (G), omics, and outcome (Y) parameters. In the M-step, replace each missing omics entry Z_ib by the closed-form solution of Equation (17): its conditional expectation under the current responsibilities and the estimated Gaussian mixture (cluster mean and covariance) parameters, using observed values in the same row while keeping observed Z_ia fixed. Iterate responsibility computation and imputation updates until the EM convergence criterion is met; the integrated approach yields consistent, less biased estimates than separate imputation in the paper's simulation studies.

## Related tools

- **LUCIDus** (R package implementing the LUCID model with the integrated EM imputation update (Eq. 17) for incomplete omics data) — https://github.com/USCbiostats/LUCIDus
- **mclust** (Default method, together with regression, for initializing LUCID model parameters before EM iteration)
- **LUCIDus3** (Extended CRAN package (3.1.0) offering missing-data diagnostics and imputation helpers (analyze_missing_pattern, check_na, safe_impute, fill_data) for incomplete omics matrices) — https://github.com/USCbiostats/LUCIDus3

## Examples

```
library(LUCIDus); data(simulated_HELIX_data); fit <- lucid(G = simulated_HELIX_data$exposure, Z = simulated_HELIX_data$omics, Y = simulated_HELIX_data$outcome, K = 2, seed = 1008)
```

## Evaluation signals

- On simulated HELIX-based data with known truth, imputed Z values are consistent and minimally biased relative to the true underlying values (paper's simulation studies)
- The EM iteration converges: responsibilities and imputed entries stabilize per the convergence criterion
- Observed entries Z_ia remain unchanged after imputation; only missing positions are updated
- Imputed values are conditional expectations under the current Gaussian mixture, so they fall within the range of cluster-mean values weighted by responsibilities
- Downstream LUCID fit recovers interpretable cluster structure (e.g., two proteomics clusters separating higher and lower body mass index)

## Limitations

- Imputation is only valid within the LUCID EM framework; Eq. 17 depends on current responsibilities and mixture parameters, so results are not a standalone imputation
- The simulation evidence is based on a realistic but simulated dataset derived from the HELIX study, not a fully observed real benchmark
- LUCIDus is actively extended in LUCIDus3; the original repository is retained for reproducibility of the published method, so version matters when reproducing Eq. 17 behavior

## Evidence

- [abstract] Integrated EM imputation update: "we extend the model to incorporate list-wise missingness within an integrated imputation framework, which can also handle sporadic missingness when necessary."
- [abstract] Simulation performance: "Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values."
- [intro] Initialization method: "The first method (also the default method) uses `mclust` and regression to initialize parameters."
- [intro] Missingness patterns considered: "We consider 2 missing patterns in omics data: (1) list-wise missing pattern that only a subset of observations have measured omics data and (2) sporadic missing pattern"
- [abstract] Package implementation and repository: "The proposed methods have been implemented in the R package LUCIDus."
- [readme] Missing-data helpers in LUCIDus3: [evidence span withheld — anchor status: not_found]
