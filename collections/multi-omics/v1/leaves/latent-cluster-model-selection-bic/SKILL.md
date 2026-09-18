---
name: latent-cluster-model-selection-bic
description: Use when fitting a LUCID model with lucid()/estimate_lucid() and the
  number of latent subgroups K is unknown and must be data-driven rather than fixed
  a priori.
when_to_use_negative:
- The number of clusters is fixed by study design or prior knowledge and K should
  not be data-driven
- The goal is plain clustering of a single data view (e.g., mclust alone) rather than
  joint estimation across exposures, omics, and outcome
- Serial LUCID models where topology selection is constrained and only scalar penalties
  are accepted
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0091
tools:
- name: LUCIDus
  role: R package providing lucid() with automatic BIC-based selection of the number
    of latent clusters K
  repo: https://github.com/USCbiostats/LUCIDus
- name: LUCIDus3
  role: Current CRAN (3.1.0) implementation offering tune_lucid() to explicitly evaluate
    a grid of K and penalty values
  repo: https://github.com/USCbiostats/LUCIDus3
- name: R
  role: Runtime environment for fitting and tuning the LUCID model
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:d9c461612ef27d0326a6fc2efb7be9031a15f9307401760c00b451ddf127bdf8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - LUCIDus
  - LUCIDus3
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# latent-cluster-model-selection-bic

## Summary

Select the optimal number of latent clusters (K) in a LUCID integrative clustering model via automatic BIC-based model selection, as demonstrated by confirming K=2 clusters in the HCB–BMI Exposome Data Challenge analysis.

## When to use

Use when fitting a LUCID model with lucid()/estimate_lucid() and the number of latent subgroups K is unknown and must be data-driven rather than fixed a priori.

## When NOT to use

- The number of clusters is fixed by study design or prior knowledge and K should not be data-driven
- The goal is plain clustering of a single data view (e.g., mclust alone) rather than joint estimation across exposures, omics, and outcome
- Serial LUCID models where topology selection is constrained and only scalar penalties are accepted

## Inputs

- Exposure matrix G (e.g., prenatal hexachlorobenzene levels or quartiles)
- Omics matrix Z (proteomics data with list-wise/sporadic missingness from 1301 children)
- Outcome vector Y (childhood BMI z-score)
- Candidate K values (integer or integer vector) and optional regularization penalty grid

## Outputs

- Optimal K selected by minimum BIC (K=2 in the HCB-BMI application)
- Fitted LUCID model object with cluster-specific parameters (mean BMI z-scores -1.78, -1.52)
- Model selection comparison (BIC across K candidates) from tune_lucid()/lucid()

## How to apply

Fit the LUCID model with lucid() in R/LUCIDus, passing the exposure matrix G (e.g., prenatal hexachlorobenzene), omics matrix Z (incomplete proteomics from 1301 children), and outcome Y (childhood BMI z-score); lucid() automatically conducts model selection on the number of clusters K, so supply K as a candidate vector instead of a scalar to trigger tuning. For finer control, use tune_lucid() to explicitly evaluate a grid of K values (and penalties for early/parallel models) and pick the model minimizing BIC. In the Exposome Data Challenge application this procedure confirmed K=2, yielding two clusters with mean BMI z-scores of -1.78 and -1.52, one of which is enriched in higher HCB quartiles (ORs 1.2, 1.71, 3.19 for quartiles 2–4). Validate the choice by checking cluster separation in summary_lucid() output and the Sankey diagram from plot_lucid(), and report bootstrap-based posterior inclusion probabilities for the selected model.

## Related tools

- **LUCIDus** (R package providing lucid() with automatic BIC-based selection of the number of latent clusters K) — https://github.com/USCbiostats/LUCIDus
- **LUCIDus3** (Current CRAN (3.1.0) implementation offering tune_lucid() to explicitly evaluate a grid of K and penalty values) — https://github.com/USCbiostats/LUCIDus3
- **R** (Runtime environment for fitting and tuning the LUCID model)

## Examples

```
library(LUCIDus); fit <- lucid(G = G_hcb, Z = Z_proteomics, Y = Y_bmi_z, K = 2:5, family = "normal", seed = 1008)
```

## Evaluation signals

- BIC is minimized at the selected K (K=2) across the candidate grid
- summary_lucid() reports two clusters with distinct mean BMI z-scores (-1.78 and -1.52)
- plot_lucid() Sankey diagram reproduces the exposure-to-cluster odds ratios 1.2, 1.71, and 3.19 for HCB quartiles 2–4
- Bootstrap-based posterior inclusion probabilities are non-degenerate for the selected model
- Fitting completes without convergence failure on the 1301-child proteomics matrix with 2.56%–37.50% missingness by HCB quartile

## Limitations

- BIC-based selection assumes the LUCID likelihood; misspecified missingness handling (list-wise vs sporadic) can bias the comparison
- Serial models accept only scalar penalties, so tuning scope is narrower than for early/parallel models
- The article's discussion section provides no extractable limitations ('No changelog found.'), so extra validation (bootstrap CIs, prediction checks) is advisable

## Evidence

- [intro] Automatic BIC selection of K within lucid(): "`lucid()` will automatically conduct model selection on number of clusters `K`, select informative variables in `G` or `Z`, or both"
- [readme] tune_lucid() explicit K grid: [evidence span withheld — anchor status: not_found]
- [readme] K input topology for early models: [evidence span withheld — anchor status: external_doc]
- [abstract] Two BMI clusters in the applied analysis: "The model successfully estimated proteomics profiles for two clusters representing higher and lower body mass index"
- [intro] Workflow step of summarization and inference after selection: "statistical inference can be accomplished for LUCID by constructing confidence intervals (CIs) based on bootstrap resampling."
