---
name: parameter-estimate-bias-and-variance-assessment
description: Use when evaluating whether an imputation or analysis method yields unbiased,
  low-variance latent-clustering parameter estimates on incomplete multi-omics data
  with known ground-truth simulation parameters.
when_to_use_negative:
- The omics data have only sporadic (scattered, entry-level) missingness rather than
  list-wise (block-wise) missingness — the L method handles sporadic missingness but
  this bias/SD benchmark targets list-wise patterns
- No ground-truth simulation parameters are available, so bias of parameter estimates
  cannot be computed against true values
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LUCIDus
  role: R package used to fit the LUCID model with integrated imputation (method L)
    and generate/analyze the simulation replicates
  repo: https://github.com/USCbiostats/LUCIDus
- name: mclust
  role: 'Provides the default initialization: mclust and regression are used to initialize
    LUCID parameters before EM estimation'
- name: R
  role: Statistical environment in which the simulation, missingness induction, model
    fitting, and bias/SD aggregation are run
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:8766b79ecf2c22cbd05a46109db6460ed9e684375d47abd9125c5fdfb614abdf
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# parameter-estimate-bias-and-variance-assessment

## Summary

A simulation-based benchmarking skill for quantifying parameter-estimate bias and standard deviations of LUCID model estimates under list-wise omics missingness (ratios 0.1–0.8), comparing the integrated imputation method (L) against complete-case analysis, impute-mclust, impute-mice, and impute-EM.

## When to use

Use when evaluating whether an imputation or analysis method yields unbiased, low-variance latent-clustering parameter estimates on incomplete multi-omics data with known ground-truth simulation parameters.

## When NOT to use

- The omics data have only sporadic (scattered, entry-level) missingness rather than list-wise (block-wise) missingness — the L method handles sporadic missingness but this bias/SD benchmark targets list-wise patterns
- No ground-truth simulation parameters are available, so bias of parameter estimates cannot be computed against true values

## Inputs

- Simulated LUCID datasets: exposure matrix G (10 exposures), omics matrix Z (4 omics variables), outcome Y, K=2 true clusters
- Train/validation split: 8000 training and 2000 validation samples per replication
- List-wise missingness ratios in omics data ranging from 0.1 to 0.8
- Fitted LUCID parameter estimates from method L and competitors (complete-case, impute-mclust, impute-mice, impute-EM) across 300 replications

## Outputs

- Per-method, per-missing-ratio parameter-estimate bias and standard deviations
- Median AUC of cluster/outcome prediction on the 2000-sample validation set
- Aggregated comparison table fig3_simulation_results.csv reproducing the Fig. 3 comparisons

## How to apply

For each of 300 replications, generate simulated data with 10 exposures, 4 omics variables, and one outcome, splitting into 8000 training and 2000 validation samples with true latent cluster number K=2. Induce list-wise missingness in the omics block of the validation set at each ratio from 0.1 to 0.8. Fit the LUCID model with integrated imputation (method L, initialized with mclust-based regression) alongside the competing methods (complete-case, impute-mclust, impute-mice, impute-EM) on each incomplete dataset. Across replications, compute the bias and standard deviations of the LUCID parameter estimates and the median AUC of cluster/outcome prediction on the 2000-sample validation set per method and missing ratio, aggregating into a fig3_simulation_results.csv. Judgment: method L estimates should stay centered on the true values with the smallest SDs and most stable, highest median AUCs, while impute-mice should show drastic bias towards 0 at missing ratios above 0.5.

## Related tools

- **LUCIDus** (R package used to fit the LUCID model with integrated imputation (method L) and generate/analyze the simulation replicates) — https://github.com/USCbiostats/LUCIDus
- **mclust** (Provides the default initialization: mclust and regression are used to initialize LUCID parameters before EM estimation)
- **R** (Statistical environment in which the simulation, missingness induction, model fitting, and bias/SD aggregation are run)

## Examples

```
library(LUCIDus); fit <- estimate_lucid(G = G, Z = Z, Y = Y, lucid_model = "early", family = "normal", K = 2, seed = 1008); summary(fit)
```

## Evaluation signals

- Method L's exposure-effect estimates remain centered on the true simulation values with the smallest standard deviations across all missing ratios 0.1–0.8
- Median AUC for method L is the most stable and highest among methods on the 2000-sample validation set at every missing ratio
- impute-mice exhibits drastic bias towards 0, especially at missing ratios above 0.5, serving as a sanity check that the simulation harness reproduces the paper's findings
- Aggregated results match the schema of fig3_simulation_results.csv: bias, SD, and median AUC per method and per missing ratio over 300 replications
- Estimated cluster number in simulated data recovers the true K=2

## Limitations

- Findings are grounded in simulated data with 10 exposures, 4 omics variables, and K=2; performance may differ for other data dimensions or cluster counts
- The LUCIDus repository README notes the original repo is retained mainly for reproducibility of the published method and recommends LUCIDus3 (CRAN LUCIDus 3.1.0) for new projects
- The article's discussion section content is absent ('No changelog found.'), so limitations beyond the simulation setup are not extractable from the provided context

## Evidence

- [abstract] Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values.: "Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values."
- [intro] We consider 2 missing patterns in omics data: (1) list-wise missing pattern...: "We consider 2 missing patterns in omics data: (1) list-wise missing pattern that only a subset of observations have measured omics data and (2) sporadic missing pattern"
- [intro] The first method (also the default method) uses `mclust` and regression to initialize parameters.: "The first method (also the default method) uses `mclust` and regression to initialize parameters."
- [intro] we use the Expectation-Maximization (EM) algorithm to obtain the maximum likelihood estimates (MLE) of LUCID.: "we use the Expectation-Maximization (EM) algorithm to obtain the maximum likelihood estimates (MLE) of LUCID."
- [readme] This repository remains available for reproducibility of the original published method.: "This repository remains available for reproducibility of the original published method."
