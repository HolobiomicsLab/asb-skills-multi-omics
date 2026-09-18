---
name: imputation-method-benchmarking
description: Use when choosing among imputation strategies for omics data with sporadic
  missingness (missing completely at random in individual entries), or when complete-case
  analysis is infeasible and bias of downstream clustering estimates must be quantified.
when_to_use_negative:
- Data contain list-wise (block-wise) missingness only and complete-case analysis
  is feasible, so no imputation comparison is needed
- The goal is predictive performance of the imputed values themselves rather than
  bias of downstream LUCID cluster/parameter estimates
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_3366
tools:
- name: LUCIDus
  role: Fits the LUCID model via lucid() with integrated imputation of missing omics
    values inside the EM algorithm; used for both the integrated-imputation arm and
    (after mean imputation) the competing arm
  repo: https://github.com/USCbiostats/LUCIDus
- name: R
  role: Language in which the simulation, missingness induction, model fitting, and
    bias tabulation are implemented
  repo: https://github.com/USCbiostats/LUCIDus
- name: mclust
  role: Used by LUCIDus as the default method to initialize parameters (mclust and
    regression) before EM estimation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:82b331e63346b16746963af6f766d24db4b32a0dedfa160a9efac2fd8410cef9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - LUCIDus
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# imputation-method-benchmarking

## Summary

A simulation-based benchmarking protocol for comparing imputation strategies (e.g., LUCID integrated imputation within the EM algorithm vs. simple mean imputation) for incomplete omics data, by measuring parameter bias against known true values across replicates.

## When to use

Use when choosing among imputation strategies for omics data with sporadic missingness (missing completely at random in individual entries), or when complete-case analysis is infeasible and bias of downstream clustering estimates must be quantified.

## When NOT to use

- Data contain list-wise (block-wise) missingness only and complete-case analysis is feasible, so no imputation comparison is needed
- The goal is predictive performance of the imputed values themselves rather than bias of downstream LUCID cluster/parameter estimates

## Inputs

- Simulated multi-omics data with known latent clusters (exposure matrix G, omics matrix Z, outcome Y)
- Sporadic missingness induced in the omics matrix (MCAR in individual omics entries)
- True underlying omics profile parameters for each cluster

## Outputs

- Bias/consistency comparison table of integrated imputation vs. impute-mean (e.g., sporadic_imputation_comparison.csv)
- Per-replicate estimated omics profile parameters for each imputation method

## How to apply

Set up a simulation configuration with a LUCID model structure: exposure matrix G, omics matrix Z with known latent clusters, and outcome Y. Simulate multi-omics data with known true omics profile parameters, then induce sporadic missingness (missingness completely at random in individual omics entries) in Z. Fit the LUCID model with lucid() using the integrated imputation framework, which imputes missing omics values within the EM algorithm; as the competing method, apply mean imputation (impute-mean) to the missing entries first and then fit the LUCID model on the completed data. Across replicates, compare estimated omics profile parameters against true underlying values and compute bias for each method, tabulating results (e.g., to sporadic_imputation_comparison.csv).

## Related tools

- **LUCIDus** (Fits the LUCID model via lucid() with integrated imputation of missing omics values inside the EM algorithm; used for both the integrated-imputation arm and (after mean imputation) the competing arm) — https://github.com/USCbiostats/LUCIDus
- **R** (Language in which the simulation, missingness induction, model fitting, and bias tabulation are implemented) — https://github.com/USCbiostats/LUCIDus
- **mclust** (Used by LUCIDus as the default method to initialize parameters (mclust and regression) before EM estimation)

## Examples

```
library(LUCIDus); fit_integrated <- lucid(G = G, Z = Z_missing, Y = Y, K = 2, seed = 1008) # vs. Z_mean <- fill_data(Z_missing); fit_mean <- lucid(G = G, Z = Z_mean, Y = Y, K = 2)
```

## Evaluation signals

- Integrated imputation yields less biased, more consistent omics profile estimates than impute-mean, matching the pattern in Supplementary section A
- Bias for each method is computed against known true omics profile parameters across simulation replicates
- Both arms are fit on identically simulated/missing data, differing only in imputation strategy (integrated EM imputation vs. prior mean imputation)
- Results are written to sporadic_imputation_comparison.csv and reproduce the direction of the published simulation findings

## Limitations

- Simulation applies to sporadic missingness (missing completely at random in individual omics entries); list-wise missingness is handled by the same framework but this benchmark targets the sporadic pattern
- The discussion section of the source document is absent ('No changelog found.'), so no explicitly stated limitations or reproducibility gaps can be extracted
- Comparison relies on simulated ground truth; real-data (e.g., HELIX proteomics) applications lack true parameter values for direct bias computation

## Evidence

- [supplementary] integrated imputation outperforms mean imputation under sporadic missingness: [evidence span withheld — anchor status: not_found]
- [intro] sporadic vs list-wise missingness definitions: "We consider 2 missing patterns in omics data: (1) list-wise missing pattern that only a subset of observations have measured omics data and (2) sporadic missing pattern that missingness is completely"
- [intro] lucid() fitting arguments: "The function `lucid()` is the primary function in the package, which fits a LUCID model based on an exposure matrix (argument `G`), omics data (argument `Z`), outcome data (argument `Y`)"
- [intro] mclust initialization: "The first method (also the default method) uses `mclust` and regression to initialize parameters."
- [abstract] EM-based imputation within LUCID: "we extend the model to incorporate list-wise missingness within an integrated imputation framework, which can also handle sporadic missingness when necessary."
