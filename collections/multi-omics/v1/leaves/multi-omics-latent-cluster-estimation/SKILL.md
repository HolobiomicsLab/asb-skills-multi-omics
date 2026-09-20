---
name: multi-omics-latent-cluster-estimation
description: Use when you need to identify latent subgroups jointly defined by an
  environmental exposure (e.g. maternal hexachlorobenzene), a multi-omics matrix (e.g.
  proteomics), and a phenotypic outcome (e.g. childhood BMI z-score), including when
  omics data have missing entries.
when_to_use_negative:
- The analysis goal is standard single-view clustering (e.g. plain mclust on Z alone)
  with no exposure-outcome integration — LUCID is designed for joint G-Z-Y modeling
- The omics data are complete and the goal is only a sensitivity comparison; a complete-case
  refit may be simpler than invoking integrated imputation
- The data structure requires multiple omics layers or multi-stage topologies that
  the original single-omics LUCIDus workflow does not cover (use the LUCIDus3 parallel/serial
  models instead)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2269
tools:
- name: LUCIDus
  role: 'R package implementing the LUCID model: lucid() fitting, BIC-based K selection,
    variable selection, missing-data handling, summary_lucid(), plot_lucid(), predict_lucid(),
    bootstrap inference'
  repo: https://github.com/USCbiostats/LUCIDus
- name: mclust
  role: 'Default initialization method: mclust plus regression initializes the EM
    parameters of the LUCID model'
- name: LUCIDus3
  role: Actively maintained extension (LUCIDus 3.1.0) providing estimate_lucid(),
    tune_lucid(), boot_lucid(), and multi-omics early/parallel/serial integration
  repo: https://github.com/USCbiostats/LUCIDus3
- name: R
  role: Runtime environment in which the LUCID model is fitted and the Exposome Data
    Challenge 2021 data are analyzed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:291371c2492782166101060ea8a6632a89ad31342edfb45f6770597d6872f6d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - LUCIDus
  - LUCIDus3
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# multi-omics-latent-cluster-estimation

## Summary

Fit a LUCID (Latent Unknown Clustering Integrating multi-omics Data) model that jointly estimates latent clusters from an exposure matrix (G), omics data (Z), and an outcome (Y), using EM estimation with mclust-based initialization, BIC selection of cluster number K, and regularization-based variable selection.

## When to use

Use when you need to identify latent subgroups jointly defined by an environmental exposure (e.g. maternal hexachlorobenzene), a multi-omics matrix (e.g. proteomics), and a phenotypic outcome (e.g. childhood BMI z-score), including when omics data have missing entries.

## When NOT to use

- The analysis goal is standard single-view clustering (e.g. plain mclust on Z alone) with no exposure-outcome integration — LUCID is designed for joint G-Z-Y modeling
- The omics data are complete and the goal is only a sensitivity comparison; a complete-case refit may be simpler than invoking integrated imputation
- The data structure requires multiple omics layers or multi-stage topologies that the original single-omics LUCIDus workflow does not cover (use the LUCIDus3 parallel/serial models instead)

## Inputs

- Exposure matrix G (e.g. maternal hexachlorobenzene levels from the ISGlobal/ATHLETE Exposome Data Challenge 2021 dataset)
- Omics data matrix Z (e.g. proteomics from 1301 children, possibly with list-wise or sporadic missingness)
- Outcome variable Y (e.g. scaled childhood BMI z-score)
- Optional covariates CoG/CoY

## Outputs

- Fitted LUCID model object with cluster assignments for K latent clusters
- Cluster-specific exposure effects, omics profiles, and outcome distributions (via summary_lucid())
- Model selection results (optimal K by BIC) and selected informative variables in G or Z
- Sankey diagram of G-Z-Y interplay (plot_lucid()) and optional bootstrap confidence intervals

## How to apply

Load the exposure matrix, omics matrix, and outcome into R and call the package's primary fitting function, passing the exposure as argument G, omics data as Z, and outcome as Y. The model is estimated via the EM algorithm; by default parameters are initialized with mclust and regression, and the function automatically conducts model selection on the number of clusters K via BIC while selecting informative variables in G or Z through regularization (lasso for exposures, graphical lasso for omics covariance). For list-wise missing omics data (e.g. incomplete proteomics measured in only a subset of the 1301 children), use the integrated imputation variant, which simulation studies showed produces consistent and less biased estimates; alternatively restrict to complete cases and refit for comparison. Summarize the fitted model with summary_lucid() to obtain cluster-specific exposure effects, omics profiles, and outcome distributions (e.g. cluster means of scaled BMI z-score), and optionally visualize with plot_lucid() Sankey diagrams, bootstrap CIs via boot_lucid(), and predictions via predict_lucid().

## Related tools

- **LUCIDus** (R package implementing the LUCID model: lucid() fitting, BIC-based K selection, variable selection, missing-data handling, summary_lucid(), plot_lucid(), predict_lucid(), bootstrap inference) — https://github.com/USCbiostats/LUCIDus
- **mclust** (Default initialization method: mclust plus regression initializes the EM parameters of the LUCID model)
- **LUCIDus3** (Actively maintained extension (LUCIDus 3.1.0) providing estimate_lucid(), tune_lucid(), boot_lucid(), and multi-omics early/parallel/serial integration) — https://github.com/USCbiostats/LUCIDus3
- **R** (Runtime environment in which the LUCID model is fitted and the Exposome Data Challenge 2021 data are analyzed)

## Examples

```
fit_early <- estimate_lucid(G = G, Z = Z, Y = Y, lucid_model = "early", family = "normal", K = 2, seed = 1008); summary(fit_early)
```

## Evaluation signals

- BIC across candidate K values selects the optimal cluster number (K = 2 for the HCB–BMI application, with clusters corresponding to higher and lower BMI)
- summary_lucid() yields interpretable cluster-specific outcome means (e.g. scaled BMI z-score μ = -1.78 in cluster 1 vs μ = -1.52 in cluster 2), ordered consistently with the finding that cluster 2 has the higher mean
- Cluster-specific omics profiles (proteomics means) and exposure effects are estimated for all K clusters without convergence warnings or numerical instability
- Results are stable under the fitted seed; bootstrap resampling produces finite confidence intervals for cluster-specific parameters
- When using integrated imputation for missing omics, estimates should be consistent and less biased than complete-case analyses, as demonstrated in the paper's simulation studies

## Limitations

- Missingness handling was developed for list-wise (block-wise) missing patterns where only a subset of observations have omics data, plus sporadic missingness when necessary; other patterns may not be covered
- Numerical instability can occur during EM fitting; the LUCIDus3 README lists stabilization helpers (safe_log_sum_exp(), check_and_stabilize_sigma(), check_convergence()) for robust workflows
- Serial models in LUCIDus3 currently accept only scalar penalties, limiting penalty tuning for multi-stage topologies

## Evidence

- [intro] lucid() fitting and inputs: "The function `lucid()` is the primary function in the package, which fits a LUCID model based on an exposure matrix (argument `G`), omics data (argument `Z`), outcome data (argument `Y`)"
- [intro] BIC model selection and variable selection: "`lucid()` will automatically conduct model selection on number of clusters `K`, select informative variables in `G` or `Z`, or both"
- [intro] mclust-based initialization: "The first method (also the default method) uses `mclust` and regression to initialize parameters."
- [abstract] Application result on HCB-BMI data: "The model successfully estimated proteomics profiles for two clusters representing higher and lower body mass index"
- [abstract] Integrated imputation performance: "Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values."
- [readme] Current package API for fitting: [evidence span withheld — anchor status: external_doc]
