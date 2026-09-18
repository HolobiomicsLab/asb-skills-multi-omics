---
name: r-package-function-implementation
description: Use when translating a published statistical algorithm (e.g., an EM imputation
  update for missing omics data) into an implemented, installable R package function
  that end users can call on their own data.
when_to_use_negative:
- The omics matrix is complete (no missing values) and only standard LUCID clustering
  without the integrated imputation I-step is needed
- The analysis goal is a standalone statistical script rather than a reusable, documented,
  installable package function with vignette and API wrappers
- The data structure is not exposures + omics + outcome (e.g., pure single-view clustering
  with no latent-cluster integration across G, Z, and Y)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3307
tools:
- name: LUCIDus
  role: R package whose source (lucid(), estimate_lucid(), tune_lucid(), safe_impute())
    serves as the implementation template for the imputation function and package
    API
  repo: https://github.com/USCbiostats/LUCIDus
- name: LUCIDus3
  role: Actively maintained CRAN extension (3.1.0) demonstrating early/parallel/serial
    model families and numerical-stability helpers to mirror in new implementations
  repo: https://github.com/USCbiostats/LUCIDus3
- name: R
  role: Implementation language for the package function and its vignette
- name: mclust
  role: Provides the default mixture-based initialization of LUCID model parameters
    before EM iteration
- name: knitr
  role: Vignette engine used to document the function workflow in the package
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:3b924d918e0fd0a1169de79435a761141e210231db97eae392896f791d7f1034
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - LUCIDus
  - LUCIDus3
  - R
  - knitr
  - mclust
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# r-package-function-implementation

## Summary

Implement a statistical method as a function inside an R package, using the LUCIDus package as a template: translate the closed-form imputation update of Equation (17) (integrated I-step for sporadic missing omics values) into package code, initialize parameters with mclust and regression, and expose the function through the package's standard workflow (fit, tune, summarize, predict).

## When to use

Use when translating a published statistical algorithm (e.g., an EM imputation update for missing omics data) into an implemented, installable R package function that end users can call on their own data.

## When NOT to use

- The omics matrix is complete (no missing values) and only standard LUCID clustering without the integrated imputation I-step is needed
- The analysis goal is a standalone statistical script rather than a reusable, documented, installable package function with vignette and API wrappers
- The data structure is not exposures + omics + outcome (e.g., pure single-view clustering with no latent-cluster integration across G, Z, and Y)

## Inputs

- Exposure matrix G (e.g., environmental exposures such as prenatal hexachlorobenzene)
- Omics data matrix Z with missing values (list-wise or sporadic patterns)
- Outcome data Y (normal or binary family)
- Optional covariate matrices CoG / CoY
- Number of latent clusters K (scalar, vector, or nested list for serial models)

## Outputs

- Fitted LUCID model object with EM estimates (responsibilities, cluster means/covariances, imputed Z entries)
- Summary tables of parameter estimates via summary_lucid()/summary()
- Predictions of cluster membership and outcome via predict_lucid()
- Sankey diagram visualization via plot_lucid()/plot()
- Bootstrap confidence intervals via boot_lucid()

## How to apply

Write the core routine as an R function operating on an exposure matrix G, omics matrix Z, and outcome Y, mirroring the `lucid()`/`estimate_lucid()` API of LUCIDus. Implement the EM loop: initialize parameters with mclust and regression (the paper's default), compute posterior cluster responsibilities in the E-step, and in the M/I-step apply the closed-form solution of Equation (17), replacing each missing omics entry Z_ib by its conditional expectation under current responsibilities and Gaussian mixture (cluster mean and covariance) parameters while leaving observed values Z_ia fixed. Handle both list-wise and sporadic missing patterns, iterate until the EM convergence criterion is met, and add numerical-stability helpers (safe_log_sum_exp, safe_normalize, safe_solve, check_and_stabilize_sigma) as the LUCIDus3 README recommends. Package the function with a vignette (knitr/rmarkdown), document installation from CRAN and GitHub via devtools, and expose wrappers for model selection (BIC over K), summary, plotting (Sankey diagram), bootstrap inference, and prediction so the function slots into the full package workflow.

## Related tools

- **LUCIDus** (R package whose source (lucid(), estimate_lucid(), tune_lucid(), safe_impute()) serves as the implementation template for the imputation function and package API) — https://github.com/USCbiostats/LUCIDus
- **LUCIDus3** (Actively maintained CRAN extension (3.1.0) demonstrating early/parallel/serial model families and numerical-stability helpers to mirror in new implementations) — https://github.com/USCbiostats/LUCIDus3
- **R** (Implementation language for the package function and its vignette)
- **mclust** (Provides the default mixture-based initialization of LUCID model parameters before EM iteration)
- **knitr** (Vignette engine used to document the function workflow in the package)

## Examples

```
install.packages("LUCIDus"); library(LUCIDus); fit <- estimate_lucid(G=G, Z=Z, Y=Y, lucid_model="early", family="normal", K=2, seed=1008)
```

## Evaluation signals

- On simulated HELIX-based data with known missing entries, imputed values from the Eq. 17 update are consistent and minimally biased relative to true underlying values, as in the paper's simulation studies
- EM loop terminates only when the convergence criterion is met and observed omics values Z_ia remain unchanged across iterations (only missing Z_ib are updated)
- Model selection over K via BIC returns the expected number of clusters (e.g., two proteomics clusters corresponding to higher and lower body mass index in the HELIX application with 1301 children)
- Package installs cleanly from CRAN (install.packages('LUCIDus')) and from GitHub (devtools::install_github('USCbiostats/LUCIDus'), optionally with build_vignettes = TRUE)
- Downstream API functions (summary_lucid, plot_lucid, predict_lucid, boot_lucid) run without error on the fitted object, confirming the function conforms to the package workflow

## Limitations

- The discussion section of the article is essentially absent ('No changelog found.'), so no extracted limitations or reproducibility gaps are available from the paper itself
- Original LUCIDus2 repository is retained only for reproducibility of the published method; the README directs new projects to LUCIDus3 for improved features and ongoing updates
- Serial models accept only scalar penalties, and penalty tuning is limited to 'early' and 'parallel' model families per the LUCIDus3 README

## Evidence

- [methods] Closed-form Eq. 17 imputation update: [evidence span withheld — anchor status: not_found]
- [intro] Default initialization with mclust and regression: "The first method (also the default method) uses `mclust` and regression to initialize parameters."
- [intro] Primary fitting function API: "The function `lucid()` is the primary function in the package, which fits a LUCID model based on an exposure matrix (argument `G`), omics data (argument `Z`), outcome data (argument `Y`)"
- [abstract] Package availability claim: "The proposed methods have been implemented in the R package LUCIDus."
- [abstract] Simulation validation of imputation: "Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values."
- [readme] Numerical-stability helpers in the package implementation: [evidence span withheld — anchor status: external_doc]
- [readme] Repository redirection for new work: "We recommend using the LUCIDus3 repository for new projects, as it includes improved features for multi-omics integration, expanded functionality, and ongoing updates."
