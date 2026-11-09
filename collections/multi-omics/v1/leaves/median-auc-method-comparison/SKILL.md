---
name: median-auc-method-comparison
description: Use when evaluating how clustering/prediction performance of methods
  for handling list-wise missing omics data degrades as missingness ratio increases,
  to select the most stable imputation-integrated clustering approach.
when_to_use_negative:
- Missingness is sporadic (element-wise) rather than list-wise, since the benchmark
  and integrated imputation scheme are designed for list-wise missing patterns in
  omics data
- The goal is only parameter-estimate bias/SD comparison rather than prediction/cluster-recovery
  performance, in which case a bias-and-SD-only design without AUC aggregation suffices
- The data contain no missingness, so no imputation-vs-complete-case comparison is
  meaningful
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0091
tools:
- name: LUCIDus
  role: R package used to fit the LUCID model with integrated imputation (method L)
    and to obtain cluster/outcome predictions for AUC computation
  repo: https://github.com/USCbiostats/LUCIDus
- name: R
  role: Statistical environment in which the simulation, missingness induction, model
    fitting, and AUC aggregation are executed
  repo: https://github.com/USCbiostats/LUCIDus
- name: mclust
  role: Used to initialize LUCID model parameters via mclust-based regression and
    as the basis of the impute-mclust competitor
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:d4ce0cbeaa394e1fa4bbec3411487b88c3aa2e56409554e88ec01b5cb6c9efea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - LUCIDus
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# median-auc-method-comparison

## Summary

A benchmarking procedure that compares imputation/clustering methods (LUCID integrated imputation 'L', complete-case, impute-mclust, impute-mice, impute-EM) by the median AUC of cluster/outcome prediction across increasing list-wise missing ratios in omics data, as used in the LUCID incomplete-omics simulation study (Zhao et al., 2024).

## When to use

Use when evaluating how clustering/prediction performance of methods for handling list-wise missing omics data degrades as missingness ratio increases, to select the most stable imputation-integrated clustering approach.

## When NOT to use

- Missingness is sporadic (element-wise) rather than list-wise, since the benchmark and integrated imputation scheme are designed for list-wise missing patterns in omics data
- The goal is only parameter-estimate bias/SD comparison rather than prediction/cluster-recovery performance, in which case a bias-and-SD-only design without AUC aggregation suffices
- The data contain no missingness, so no imputation-vs-complete-case comparison is meaningful

## Inputs

- Simulated or real multi-omics data: exposure matrix G, omics matrix Z, outcome Y with a known or assumed latent cluster count K
- List-wise missingness ratio grid applied to the omics validation block (0.1 to 0.8)
- R environment with the LUCIDus package installed from USCbiostats/LUCIDus
- Replication-level AUC values from cluster/outcome prediction on a held-out validation set

## Outputs

- Median AUC per imputation/clustering method (L, complete-case, impute-mclust, impute-mice, impute-EM) per missing ratio
- Comparison table such as fig3_simulation_results.csv reproducing the Fig. 3 AUC comparisons
- Accompanying parameter-estimate bias and standard-deviation summaries for each method

## How to apply

Generate replicated simulated datasets with an exposures matrix (G, e.g. 10 exposures), an omics matrix (Z, e.g. 4 omics variables), and an outcome (Y), with a known true latent cluster number K=2. For each replication, induce list-wise missingness in the omics block of a held-out validation set (e.g. 2000 samples from 10000 total) at each missing ratio from 0.1 to 0.8. Fit the LUCID model with integrated imputation (method L, initialized with mclust-based regression) alongside the competitors — complete-case analysis, impute-mclust, impute-mice, impute-EM — using the LUCIDus R package. On each incomplete validation set, obtain predicted cluster assignments and outcome predictions (predict_lucid()) and compute the AUC of cluster/outcome prediction; aggregate across the 300 replications by taking the median AUC per method per missing ratio. Judge correctness by whether the L method's median AUC remains the highest and most stable as ratios rise, while impute-mice collapses toward 0 bias at ratios above 0.5; tabulate results per method and ratio (as in fig3_simulation_results.csv) alongside parameter-estimate bias and standard deviations.

## Related tools

- **LUCIDus** (R package used to fit the LUCID model with integrated imputation (method L) and to obtain cluster/outcome predictions for AUC computation) — https://github.com/USCbiostats/LUCIDus
- **R** (Statistical environment in which the simulation, missingness induction, model fitting, and AUC aggregation are executed) — https://github.com/USCbiostats/LUCIDus
- **mclust** (Used to initialize LUCID model parameters via mclust-based regression and as the basis of the impute-mclust competitor)

## Examples

```
library(LUCIDus); fit <- lucid(G = G_train, Z = Z_train, Y = Y_train, K = 2); pred <- predict_lucid(fit, G = G_valid, Z = Z_valid, Y = Y_valid); auc <- compute_auc(Y_valid, pred$posterior)
```

## Evaluation signals

- Median AUC for the L method remains the highest and most stable across missing ratios 0.1–0.8, consistent with the paper's Fig. 3 finding
- impute-mice's exposure-effect estimates are drastically biased toward 0, especially at missing ratios above 0.5
- Each of the 300 replications uses 8000 training and 2000 validation samples with true K=2, and AUC is computed on the 2000-sample validation set
- Results table (e.g. fig3_simulation_results.csv) contains one bias/SD/median-AUC row per method per missing-ratio level
- Parameter estimates from the L method stay centered on the true values with the smallest standard deviations

## Limitations

- AUC and bias conclusions are specific to the simulated setting (10 exposures, 4 omics variables, K=2, 300 replications) and may not transfer to other data structures or higher omics dimensionality
- LUCIDus README notes that this repository is retained 'for reproducibility of the original published method' and recommends the actively maintained LUCIDus3 (v3.1.0) for new projects
- The article's discussion section is absent ('No changelog found.'), so stated limitations and reproducibility gaps cannot be verified from the discussion text

## Evidence

- [results] L method's exposure-effect estimates stay centered on the true value with the smallest SDs and the most stable, highest median AUCs: [evidence span withheld — anchor status: not_found]
- [results] impute-mice biased towards 0 at high missing ratios: [evidence span withheld — anchor status: not_found]
- [intro] missing patterns in omics data: "We consider 2 missing patterns in omics data: (1) list-wise missing pattern that only a subset of observations have measured omics data"
- [readme] LUCIDus package implements the statistical method: "The **LUCIDus** package implements the statistical method LUCID proposed
in the research paper [A Latent Unknown Clustering Integrating
Multi-Omics Data (LUCID) with Phenotypic"
- [intro] mclust default initialization: "The first method (also the default method) uses `mclust` and regression to initialize parameters."
