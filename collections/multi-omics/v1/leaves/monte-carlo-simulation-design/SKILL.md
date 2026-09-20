---
name: monte-carlo-simulation-design
description: Use when you must quantify how a latent-clustering or integrative-omics
  method degrades as list-wise missingness increases, by simulating data with known
  truth and benchmarking imputation competitors.
when_to_use_negative:
- The analysis goal is applying LUCID to a real cohort dataset (e.g., the ISGlobal/ATHLETE
  HELIX exposome-proteomics application with 1301 children) rather than benchmarking
  imputation performance under controlled missingness
- Missingness in the data is not in the omics block Z — the simulation framework targets
  list-wise (and sporadic) missing patterns in omics data specifically
- You need a quick sensitivity check rather than a 300-replication benchmark; the
  full design is computationally expensive (5 methods × 8 missing ratios × 300 replications)
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3053
tools:
- name: LUCIDus
  role: R package that fits the LUCID model with integrated imputation (method L)
    and supports the missing-data handling benchmarked in the simulation
  repo: https://github.com/USCbiostats/LUCIDus
- name: R
  role: Language/environment in which the LUCIDus package and the 300-replication
    simulation are implemented
  repo: https://github.com/USCbiostats/LUCIDus
- name: mclust
  role: Used (with regression) as the default initialization for LUCID parameters
    and as the basis of the impute-mclust competitor
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:5999f6001cba712d788d92f4e5ffcf3deffed445a8d656b84d7de4e17665af0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - LUCIDus
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# monte-carlo-simulation-design

## Summary

Design a replicated Monte Carlo benchmark that induces list-wise missingness in an omics block (ratios 0.1–0.8) and compares the LUCID integrated-imputation method ('L') against complete-case analysis, impute-mclust, impute-mice, and impute-EM on estimate bias, standard deviations, and clustering AUC.

## When to use

Use when you must quantify how a latent-clustering or integrative-omics method degrades as list-wise missingness increases, by simulating data with known truth and benchmarking imputation competitors.

## When NOT to use

- The analysis goal is applying LUCID to a real cohort dataset (e.g., the ISGlobal/ATHLETE HELIX exposome-proteomics application with 1301 children) rather than benchmarking imputation performance under controlled missingness
- Missingness in the data is not in the omics block Z — the simulation framework targets list-wise (and sporadic) missing patterns in omics data specifically
- You need a quick sensitivity check rather than a 300-replication benchmark; the full design is computationally expensive (5 methods × 8 missing ratios × 300 replications)

## Inputs

- Simulated multi-view data per replication: exposure matrix G (10 exposures), omics matrix Z (4 omics variables), outcome Y
- Sample split specification: 8000 training samples and 2000 validation samples
- True latent cluster number K = 2
- Grid of list-wise missing ratios for the omics block: 0.1, 0.2, ..., 0.8

## Outputs

- fig3_simulation_results.csv with bias, standard deviation, and median AUC per method (L, complete-case, impute-mclust, impute-mice, impute-EM) and missing ratio
- Fig. 3-style comparison of parameter-estimate bias/SD and median clustering AUC across missing ratios

## How to apply

In R with the LUCIDus package, generate 300 replications of simulated data with 10 exposures (G), 4 omics variables (Z), and one outcome (Y), using 8000 training and 2000 validation samples with a true latent cluster number K=2. For each replication, induce list-wise missingness in the omics block of the validation set at each ratio from 0.1 to 0.8. On each incomplete dataset, fit the LUCID model with integrated imputation (method L, initialized with mclust-based regression) plus four competitors: complete-case analysis, impute-mclust, impute-mice, and impute-EM (the latter reflecting LUCID's EM-algorithm estimation of MLEs). Across replications, compute parameter-estimate bias and standard deviations of the LUCID parameter estimates and the median AUC of cluster/outcome prediction on the 2000-sample validation set, then aggregate bias, SD, and median AUC per method and missing ratio into fig3_simulation_results.csv reproducing the Fig. 3 comparisons. The rationale is that simulation gives known true values, so bias and AUC can be assessed directly rather than inferred.

## Related tools

- **LUCIDus** (R package that fits the LUCID model with integrated imputation (method L) and supports the missing-data handling benchmarked in the simulation) — https://github.com/USCbiostats/LUCIDus
- **R** (Language/environment in which the LUCIDus package and the 300-replication simulation are implemented) — https://github.com/USCbiostats/LUCIDus
- **mclust** (Used (with regression) as the default initialization for LUCID parameters and as the basis of the impute-mclust competitor)

## Examples

```
install.packages("LUCIDus"); library(LUCIDus); fit <- estimate_lucid(G=G, Z=Z, Y=Y, lucid_model="early", family="normal", K=2, seed=1008)
```

## Evaluation signals

- fig3_simulation_results.csv exists and contains one aggregated row per method (L, complete-case, impute-mclust, impute-mice, impute-EM) per missing ratio from 0.1 to 0.8, with columns for bias, SD, and median AUC
- Across all missing ratios, the L method's exposure-effect estimates stay centered on the true simulated values with the smallest standard deviations
- The impute-mice competitor's estimates become drastically biased towards 0, especially at missing ratios above 0.5
- The L method attains the most stable, highest median AUC of cluster/outcome prediction on the 2000-sample validation set at every missing ratio
- Results are aggregated over the full 300 replications, with the true K=2 latent structure recoverable from the fitted models

## Limitations

- Benchmark conclusions depend on the simulation configuration (10 exposures, 4 omics variables, K=2, 8000/2000 train/validation split); other data dimensions or cluster counts may not reproduce the same ranking
- impute-mice degrades sharply only at high missing ratios (above 0.5), so low-ratio results should not be extrapolated
- The article's discussion section is absent ('No changelog found.'), so no explicit limitations or reproducibility gaps are extractable from the paper itself

## Evidence

- [abstract] Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values.: "Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values."
- [intro] Two missing patterns in omics data: "We consider 2 missing patterns in omics data: (1) list-wise missing pattern that only a subset of observations have measured omics data"
- [intro] mclust initialization: "The first method (also the default method) uses `mclust` and regression to initialize parameters."
- [intro] EM algorithm estimation: "we use the Expectation-Maximization (EM) algorithm to obtain the maximum likelihood estimates (MLE) of LUCID."
- [readme] LUCIDus integrates multi-view data: "LUCID conducts integrated clustering by using multi-view data, including exposures, omics data with/without outcome."
