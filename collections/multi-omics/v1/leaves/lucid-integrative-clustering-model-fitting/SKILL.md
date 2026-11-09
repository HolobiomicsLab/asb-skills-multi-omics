---
name: lucid-integrative-clustering-model-fitting
description: Use when jointly clustering individuals on exposures (G), omics (Z),
  and an outcome (Y) — e.g. prenatal hexachlorobenzene (HCB) + proteomics + childhood
  BMI z-score — especially when the omics block contains list-wise or sporadic missing
  values.
when_to_use_negative:
- When the data lacks one of the three required views — LUCID is a joint model over
  exposures G, omics Z, and outcome Y, not a standalone omics-only clustering method
- When the omics block is complete and the goal is purely exploratory single-view
  clustering, where a standard mixture model such as plain mclust is simpler
- When the analysis plan requires cluster inference for sporadic missingness via external
  imputation-by-dropping (complete-case), which is infeasible under sporadic missing
  patterns
edam_operation: http://edamontology.org/operation_3432
tools:
- name: LUCIDus
  role: Primary R package; lucid()/estimate_lucid() fit the LUCID model, summary_lucid()
    summarizes, predict_lucid() predicts, plot_lucid() draws Sankey diagrams, boot-based
    CIs for inference
  repo: https://github.com/USCbiostats/LUCIDus
- name: LUCIDus3
  role: Current CRAN (3.1.0) maintenance repository extending multi-omics integration,
    tuning, and missing-data handling
  repo: https://github.com/USCbiostats/LUCIDus3
- name: mclust
  role: Default initialization of LUCID parameters via mclust and regression before
    EM iterations
- name: R
  role: Runtime environment in which the LUCIDus package is installed and run
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:87bca20b67d0f01b1c890979218e4c176aba8ca96d0a6cccc8637770f1cb4b4d
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# lucid-integrative-clustering-model-fitting

## Summary

Fit a LUCID (Latent Unknown Clustering Integrating Multi-Omics Data) model that jointly estimates latent subgroups from an exposure matrix G, omics matrix Z, and outcome Y, using the LUCIDus R package with EM estimation, mclust-based initialization, BIC model selection over K, and integrated imputation for incomplete omics data.

## When to use

Use when jointly clustering individuals on exposures (G), omics (Z), and an outcome (Y) — e.g. prenatal hexachlorobenzene (HCB) + proteomics + childhood BMI z-score — especially when the omics block contains list-wise or sporadic missing values.

## When NOT to use

- When the data lacks one of the three required views — LUCID is a joint model over exposures G, omics Z, and outcome Y, not a standalone omics-only clustering method
- When the omics block is complete and the goal is purely exploratory single-view clustering, where a standard mixture model such as plain mclust is simpler
- When the analysis plan requires cluster inference for sporadic missingness via external imputation-by-dropping (complete-case), which is infeasible under sporadic missing patterns

## Inputs

- Exposure matrix G (e.g. 10 simulated exposures or maternal HCB levels)
- Omics matrix Z (e.g. proteomics from 1301 children, possibly with list-wise or sporadic missingness)
- Outcome vector Y (e.g. scaled childhood BMI z-score; normal or binary family in LUCIDus 3.x)
- Optional covariates CoG and CoY
- Candidate number(s) of latent clusters K (e.g. K=2)

## Outputs

- Fitted LUCID model object with EM-estimated parameters (exposure effects, cluster-specific omics means/covariances, outcome distributions)
- BIC-selected optimal K and selected informative variables in G and/or Z
- summary_lucid() parameter/interpretation tables
- predict_lucid() cluster assignments and outcome predictions
- Bootstrap confidence intervals for parameters; Sankey diagram via plot_lucid()

## How to apply

Call lucid() (or estimate_lucid() in LUCIDus 3.x) with the exposure matrix G, omics matrix Z, and outcome Y; the package fits the model by Expectation-Maximization to obtain MLEs, with the default initialization using mclust and regression. Let lucid() automatically conduct model selection on the number of clusters K via BIC and optionally perform variable selection via regularization (lasso on exposures, graphical lasso on omics covariance). Missing omics values are imputed within the EM algorithm for sporadic patterns; for list-wise missingness use the integrated-imputation extension rather than impute-mice, which the simulations show becomes drastically biased toward 0 above 0.5 missing ratio. Summarize parameter estimates with summary_lucid(), predict cluster membership and outcomes with predict_lucid(), and quantify uncertainty with bootstrap confidence intervals.

## Related tools

- **LUCIDus** (Primary R package; lucid()/estimate_lucid() fit the LUCID model, summary_lucid() summarizes, predict_lucid() predicts, plot_lucid() draws Sankey diagrams, boot-based CIs for inference) — https://github.com/USCbiostats/LUCIDus
- **LUCIDus3** (Current CRAN (3.1.0) maintenance repository extending multi-omics integration, tuning, and missing-data handling) — https://github.com/USCbiostats/LUCIDus3
- **mclust** (Default initialization of LUCID parameters via mclust and regression before EM iterations)
- **R** (Runtime environment in which the LUCIDus package is installed and run)

## Examples

```
fit_early <- estimate_lucid(G = G, Z = Z, Y = Y, CoY = CoY, lucid_model = "early", family = "normal", K = 2, seed = 1008)
```

## Evaluation signals

- BIC-based model selection recovers the true number of latent clusters (K=2 in the paper's simulations)
- Exposure-effect estimates remain centered on the true value with the smallest standard deviations versus complete-case, impute-mclust, impute-mice, and impute-EM across missing ratios 0.1–0.8
- predict_lucid() achieves the most stable, highest median AUC of cluster/outcome prediction on the 2000-sample validation set
- summary_lucid() yields interpretable cluster-specific omics profiles distinguishing higher vs lower BMI (μBMI Cluster 1 = -1.78 vs Cluster 2 = -1.52 in the HCB–BMI Model 1 reference)
- Integrated imputation produces consistent, less biased omics profile parameters than impute-mean under sporadic missingness (Supplementary section A)

## Limitations

- Fitting depends on initialization (mclust + regression by default); poor initialization can affect EM convergence to the MLE
- Sporadic missingness is handled by imputation within the EM algorithm, while list-wise missingness requires the integrated-imputation extension — the two patterns are treated differently
- Competing external imputations are unreliable under heavy missingness: impute-mice becomes drastically biased toward 0 above a 0.5 missing ratio
- Discussion-section limitations and a changelog are absent from the extracted article text ('No changelog found.')

## Evidence

- [intro] lucid() is the primary fitting entry point taking G, Z, Y: "The function `lucid()` is the primary function in the package, which fits a LUCID model based on an exposure matrix (argument `G`), omics data (argument `Z`), outcome data (argument `Y`)"
- [intro] mclust-based default initialization: "The first method (also the default method) uses `mclust` and regression to initialize parameters."
- [intro] EM algorithm for maximum likelihood estimation: "we use the Expectation-Maximization (EM) algorithm to obtain the maximum likelihood estimates (MLE) of LUCID."
- [intro] automatic model selection over K: "`lucid()` will automatically conduct model selection on number of clusters `K`, select informative variables in `G` or `Z`, or both"
- [readme] LUCIDus3 scope statement: [evidence span withheld — anchor status: not_found]
- [other] Integrated imputation outperforms competitors under list-wise missingness: [evidence span withheld — anchor status: not_found]
