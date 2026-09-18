---
name: multi-omics-integrative-clustering
description: Use when you have an exposure matrix, an incomplete omics matrix, and
  a phenotypic outcome (e.g., prenatal hexachlorobenzene, proteomics with list-wise/sporadic
  missingness, childhood BMI z-score) and want latent clusters linking them.
when_to_use_negative:
- When the omics matrix is complete with no missingness and the goal is ordinary clustering
  unrelated to exposures/outcome — a standard mixture model (e.g., mclust alone) suffices
  without the LUCID G-Z-Y joint structure
- When the research question is direct exposure-outcome regression without latent
  subgroups — LUCID's cluster-mediated G-Z-Y structure is unnecessary
- When omics layers require the newer parallel/serial multi-layer topologies and you
  are restricted to the original single-omics LUCIDus 2.x workflow (use LUCIDus 3.1.0
  instead)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0622
tools:
- name: LUCIDus
  role: Primary R package fitting the LUCID model via lucid(), with mclust-based initialization,
    integrated imputation for missing omics, BIC model selection, summary_lucid(),
    plot_lucid() Sankey diagrams, and bootstrap inference
  repo: https://github.com/USCbiostats/LUCIDus
- name: LUCIDus3
  role: Actively maintained extension (LUCIDus 3.1.0) supporting early/parallel/serial
    multi-omics integration, estimate_lucid(), tune_lucid(), boot_lucid(), and missing-data
    diagnostics/imputation helpers
  repo: https://github.com/USCbiostats/LUCIDus3
- name: mclust
  role: Used with regression to initialize LUCID model parameters (default initialization
    method, Algorithm 1)
- name: R
  role: Runtime environment for LUCIDus and the analysis workflow
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:eb751eb92dcc8ba9195e7ec127d2ca901d1d2c56aab9104c1b71183aefb4bc9e
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
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# multi-omics-integrative-clustering

## Summary

Fit a LUCID (Latent Unknown Clustering Integrating Multi-omics Data) model that jointly estimates latent clusters from exposures (G), omics data (Z), and an outcome (Y), using an EM algorithm with integrated imputation to handle list-wise and sporadic missing omics values, as implemented in the R package LUCIDus.

## When to use

Use when you have an exposure matrix, an incomplete omics matrix, and a phenotypic outcome (e.g., prenatal hexachlorobenzene, proteomics with list-wise/sporadic missingness, childhood BMI z-score) and want latent clusters linking them.

## When NOT to use

- When the omics matrix is complete with no missingness and the goal is ordinary clustering unrelated to exposures/outcome — a standard mixture model (e.g., mclust alone) suffices without the LUCID G-Z-Y joint structure
- When the research question is direct exposure-outcome regression without latent subgroups — LUCID's cluster-mediated G-Z-Y structure is unnecessary
- When omics layers require the newer parallel/serial multi-layer topologies and you are restricted to the original single-omics LUCIDus 2.x workflow (use LUCIDus 3.1.0 instead)

## Inputs

- Exposure matrix G (e.g., prenatal hexachlorobenzene levels or HCB quartiles)
- Omics matrix Z with list-wise and/or sporadic missing patterns (e.g., proteomics from 1301 children, incomplete)
- Outcome Y (e.g., childhood BMI z-score; normal or binary family supported)
- Optional covariates (CoY/CoG)
- Number of latent clusters K (integer or grid for tuning)

## Outputs

- Fitted LUCID model object with cluster-specific Gaussian mixture parameters (means, covariances) and exposure-effect estimates
- Cluster assignments (posterior responsibilities) and selected K via BIC
- summary_lucid() tables of parameter estimates, cluster outcome means (e.g., BMI z-scores -1.78, -1.52), and bootstrap-based posterior inclusion probabilities
- Sankey diagram from plot_lucid() showing G-Z-Y interplay and exposure-to-cluster odds ratios (e.g., 1.2, 1.71, 3.19)
- Imputed omics matrix with missing Z entries filled by the Eq. 17 EM update
- Bootstrap confidence intervals and predictions of cluster membership and outcome

## How to apply

Assemble the exposure matrix (G), omics matrix (Z), and outcome (Y), e.g., prenatal hexachlorobenzene, proteomics from 1301 children in the ISGlobal/ATHLETE Exposome Data Challenge data, and childhood BMI z-score. Fit the model with lucid() (or estimate_lucid() in LUCIDus 3.x), initializing parameters with the default mclust-and-regression approach (Algorithm 1); the EM algorithm alternates E-step posterior cluster responsibilities with M-step maximization, and missing omics entries are updated each iteration via the closed-form conditional expectation of Eq. 17 under the current responsibilities and Gaussian mixture (cluster mean/covariance) parameters, with observed values Z_ia held fixed. Select the optimal number of clusters K automatically via BIC (the application confirmed K=2); regularization (lasso on exposures, graphical lasso on omics covariance) selects informative variables. Summarize with summary_lucid() to obtain cluster-specific outcome means (e.g., BMI z-scores -1.78 and -1.52) and bootstrap-based posterior inclusion probabilities, and visualize the G-Z-Y interplay with plot_lucid() as a Sankey diagram showing exposure-to-cluster odds ratios (e.g., HCB quartile ORs 1.2, 1.71, 3.19). LUCIDus 3.x additionally supports early, parallel, and serial multi-omics topologies, tune_lucid() for K/penalty grids, predict_lucid() for cluster/outcome prediction, and boot_lucid() for bootstrap confidence intervals.

## Related tools

- **LUCIDus** (Primary R package fitting the LUCID model via lucid(), with mclust-based initialization, integrated imputation for missing omics, BIC model selection, summary_lucid(), plot_lucid() Sankey diagrams, and bootstrap inference) — https://github.com/USCbiostats/LUCIDus
- **LUCIDus3** (Actively maintained extension (LUCIDus 3.1.0) supporting early/parallel/serial multi-omics integration, estimate_lucid(), tune_lucid(), boot_lucid(), and missing-data diagnostics/imputation helpers) — https://github.com/USCbiostats/LUCIDus3
- **mclust** (Used with regression to initialize LUCID model parameters (default initialization method, Algorithm 1))
- **R** (Runtime environment for LUCIDus and the analysis workflow)

## Examples

```
library(LUCIDus); fit <- lucid(G = exposure, Z = proteomics, Y = bmi_z, K = 2); summary_lucid(fit); plot_lucid(fit)
```

## Evaluation signals

- BIC-based selection returns a defensible K (e.g., K=2 in the HCB-BMI application) and the fitted model converges under the EM criterion
- Imputed omics values from the Eq. 17 update are consistent and minimally biased relative to true underlying values, as verified in the paper's simulation studies on the HELIX-based simulated dataset
- summary_lucid() yields plausible cluster-specific outcome means (e.g., BMI z-scores -1.78 and -1.52 for the two clusters) and interpretable PIPs
- plot_lucid() Sankey diagram reproduces coherent exposure-to-cluster odds ratios (HCB quartile ORs 1.2, 1.71, 3.19) with the higher-exposure quartile mapping to the higher-BMI cluster
- Bootstrap resampling produces finite confidence intervals for the parameter estimates

## Limitations

- List-wise missingness was the primary target; sporadic missingness is handled 'when necessary', and block-wise missing patterns beyond these two are not described
- Regularization-based variable selection (lasso for exposures, graphical lasso for omics covariance) and penalty tuning apply only to early/parallel models in LUCIDus 3.x; serial models accept scalar penalties
- The imputation framework relies on the Gaussian mixture assumption within clusters, so imputation quality degrades when cluster-conditional omics distributions are strongly non-Gaussian
- Discussion content on limitations/reproducibility gaps is absent from the extracted article text ('No changelog found.')

## Evidence

- [abstract] Integrated imputation framework: "we extend the model to incorporate list-wise missingness within an integrated imputation framework, which can also handle sporadic missingness when necessary."
- [intro] mclust initialization: "The first method (also the default method) uses `mclust` and regression to initialize parameters."
- [intro] EM imputation of missing omics: "we use the Expectation-Maximization (EM) algorithm to obtain the maximum likelihood estimates (MLE) of LUCID."
- [intro] Workflow functions: "`summary_lucid()` summarizes the fitted LUCID model by producing summary tables of parameter estimation and interpretation."
- [intro] LUCIDus features: "**LUCIDus** features variable selection, incorporating missingness in omics data, visualization of LUCID model via Sankey diagram, bootstrap inf"
- [results] Simulation performance: "Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values."
- [readme] LUCIDus3 integration strategies: [evidence span withheld — anchor status: external_doc]
- [readme] Missing-data tooling in LUCIDus3: [evidence span withheld — anchor status: external_doc]
