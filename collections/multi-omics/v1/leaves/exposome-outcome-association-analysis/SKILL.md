---
name: exposome-outcome-association-analysis
description: Use when analyzing mother-child cohort data (e.g., ISGlobal/ATHLETE Exposome
  Data Challenge, 1301 children) where an environmental exposure, an omics matrix
  with missing values, and a continuous outcome should be integrated to estimate latent
  subgroups and exposure-to-cluster-to-outcome pathways.
when_to_use_negative:
- When the omics matrix Z is complete with no list-wise or sporadic missingness —
  the imputation machinery of the extended model is unnecessary and the original LUCID
  workflow suffices
- When the goal is a direct exposure-outcome regression without latent cluster structure
  or omics mediation — a standard regression model is the appropriate tool
- When multiple omics layers must be integrated in parallel or serial topologies —
  use LUCIDus3's parallel/serial estimate_lucid() models instead of the single early-integration
  workflow
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0625
tools:
- name: LUCIDus
  role: R package that fits the LUCID model via lucid(), performs BIC-based K selection,
    integrated imputation of missing omics, summary_lucid(), bootstrap inference,
    and Sankey visualization via plot_lucid()
  repo: https://github.com/USCbiostats/LUCIDus
- name: LUCIDus3
  role: Actively maintained extension (CRAN LUCIDus 3.1.0) supporting early/parallel/serial
    integration, missing-data diagnostics (analyze_missing_pattern, safe_impute),
    tune_lucid(), predict_lucid(), and boot_lucid()
  repo: https://github.com/USCbiostats/LUCIDus3
- name: mclust
  role: 'Default initialization method: mclust-based clustering plus regression initializes
    LUCID model parameters before EM estimation'
- name: ExposomeDataChallenge2021
  role: Source repository of the ISGlobal/ATHLETE Exposome Data Challenge mother-child
    cohort data (1301 pairs) used as analysis input
  repo: https://github.com/isglobal-exposomeHub/ExposomeDataChallenge2021
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:a64def65415f76c1006aded7e41d5f1aa6cc3b4a2b543a7901e3637a5ea25c53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - ExposomeDataChallenge2021
  - LUCIDus
  - LUCIDus3
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# exposome-outcome-association-analysis

## Summary

Fit a LUCID integrative clustering model linking prenatal exposures (e.g., hexachlorobenzene), omics data with missingness, and child health outcomes (BMI z-score) to discover latent clusters that mediate exposome-outcome associations, using the LUCIDus R package.

## When to use

Use when analyzing mother-child cohort data (e.g., ISGlobal/ATHLETE Exposome Data Challenge, 1301 children) where an environmental exposure, an omics matrix with missing values, and a continuous outcome should be integrated to estimate latent subgroups and exposure-to-cluster-to-outcome pathways.

## When NOT to use

- When the omics matrix Z is complete with no list-wise or sporadic missingness — the imputation machinery of the extended model is unnecessary and the original LUCID workflow suffices
- When the goal is a direct exposure-outcome regression without latent cluster structure or omics mediation — a standard regression model is the appropriate tool
- When multiple omics layers must be integrated in parallel or serial topologies — use LUCIDus3's parallel/serial estimate_lucid() models instead of the single early-integration workflow

## Inputs

- Mother-child cohort data from isglobal-exposomeHub/ExposomeDataChallenge2021 (1301 pairs with list-wise missing proteomics)
- Exposure matrix G (prenatal hexachlorobenzene, HCB quartiles)
- Omics matrix Z (proteomics, partially missing)
- Outcome vector Y (childhood BMI z-score)

## Outputs

- Fitted LUCID model with BIC-selected number of latent clusters (K=2)
- Cluster-specific mean BMI z-scores and exposure-to-cluster odds ratios (e.g., 1.2, 1.71, 3.19 by HCB quartile)
- lucid_model_summary.csv with parameter estimates and bootstrap-based posterior inclusion probabilities
- pip_histogram.png and sankey_diagram.png showing the G-Z-Y interplay

## How to apply

Load the ExposomeDataChallenge2021 mother-child data and assemble the exposure matrix G (prenatal HCB, optionally quartile-coded), omics matrix Z (proteomics with list-wise missingness), and outcome Y (childhood BMI z-score). Fit the model with lucid() in R using mclust-and-regression initialization and the integrated imputation framework that handles list-wise missing proteomics; the function automatically selects the optimal number of clusters K via BIC (confirmed K=2 in this analysis). Summarize the fit with summary_lucid() to obtain cluster-specific mean BMI z-scores (-1.78, -1.52) and bootstrap-based posterior inclusion probabilities, and visualize the G-Z-Y interplay with plot_lucid() to extract exposure-to-cluster odds ratios (1.2, 1.71, 3.19 for HCB quartiles 2-4). Correct application is judged by BIC-based K selection, sensible cluster-outcome separation, and reproducible ORs and PIPs.

## Related tools

- **LUCIDus** (R package that fits the LUCID model via lucid(), performs BIC-based K selection, integrated imputation of missing omics, summary_lucid(), bootstrap inference, and Sankey visualization via plot_lucid()) — https://github.com/USCbiostats/LUCIDus
- **LUCIDus3** (Actively maintained extension (CRAN LUCIDus 3.1.0) supporting early/parallel/serial integration, missing-data diagnostics (analyze_missing_pattern, safe_impute), tune_lucid(), predict_lucid(), and boot_lucid()) — https://github.com/USCbiostats/LUCIDus3
- **mclust** (Default initialization method: mclust-based clustering plus regression initializes LUCID model parameters before EM estimation)
- **ExposomeDataChallenge2021** (Source repository of the ISGlobal/ATHLETE Exposome Data Challenge mother-child cohort data (1301 pairs) used as analysis input) — https://github.com/isglobal-exposomeHub/ExposomeDataChallenge2021

## Examples

```
library(LUCIDus); fit <- lucid(G = hcb_exposure, Z = proteomics, Y = bmi_zscore); summary_lucid(fit); plot_lucid(fit)
```

## Evaluation signals

- BIC-based automatic selection of the number of latent clusters K converges to K=2
- summary_lucid() reports cluster-specific mean BMI z-scores that are clearly separated (approximately -1.78 and -1.52)
- Reproduced exposure-to-cluster odds ratios by HCB quartile are close to 1.2 (Q2), 1.71 (Q3), and 3.19 (Q4), showing a monotonic dose-response
- Bootstrap-based posterior inclusion probabilities are stable, and the PIP histogram annotated with HCB-quartile missingness ratios (2.56%-37.50%) is generated
- plot_lucid() produces a Sankey diagram correctly tracing G -> Z -> Y (exposure, proteomics clusters, BMI)

## Limitations

- A significant challenge in multi-omics analysis is list-wise missingness; the integrated imputation framework addresses list-wise and sporadic missing patterns but imputation quality depends on the missingness ratio
- Simulation studies support consistency and reduced bias, but performance under very high missingness (e.g., HCB-quartile missingness ratios up to 37.50%) should be inspected via the PIP histogram annotation
- The article's discussion section is absent ('No changelog found.'), so limitations and reproducibility gaps are not exhaustively documented in the source

## Evidence

- [abstract] to explore the association between maternal exposure to hexachlorobenzene and childhood body mass index by integrating incomplete proteomics data from 1301 children: "to explore the association between maternal exposure to hexachlorobenzene and childhood body mass index by integrating incomplete proteomics data from 1301 children"
- [intro] The function lucid() is the primary function in the package, which fits a LUCID model based on an exposure matrix (argument G), omics data (argument Z), outcome data (argument Y): "The function `lucid()` is the primary function in the package, which fits a LUCID model based on an exposure matrix (argument `G`), omics data (argument `Z`), outcome data (argument `Y`)"
- [intro] The first method (also the default method) uses mclust and regression to initialize parameters.: "The first method (also the default method) uses `mclust` and regression to initialize parameters."
- [intro] lucid() will automatically conduct model selection on number of clusters K, select informative variables in G or Z, or both: "`lucid()` will automatically conduct model selection on number of clusters `K`, select informative variables in `G` or `Z`, or both"
- [intro] Visualization is performed via the plot_lucid() function to create a Sankey diagram showing the interplay among the three components (G, Z and Y).: "Visualization is performed via the `plot_lucid()` function to create a Sankey diagram showing the interplay among the three components (`G`, `Z` and `Y`)."
- [abstract] Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values.: "Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values."
- [intro] statistical inference can be accomplished for LUCID by constructing confidence intervals (CIs) based on bootstrap resampling.: "statistical inference can be accomplished for LUCID by constructing confidence intervals (CIs) based on bootstrap resampling."
- [intro] We consider 2 missing patterns in omics data: (1) list-wise missing pattern that only a subset of observations have measured omics data: "We consider 2 missing patterns in omics data: (1) list-wise missing pattern that only a subset of observations have measured omics data"
