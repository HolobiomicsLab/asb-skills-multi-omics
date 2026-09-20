---
name: posterior-inclusion-probability-visualization
description: Use when a fitted LUCIDus model (e.g., HCB exposure, incomplete proteomics,
  childhood BMI z-score) yields bootstrap-based posterior inclusion probabilities
  and you need to visualize and interpret variable inclusion uncertainty.
when_to_use_negative:
- The model has not been fitted with LUCIDus (no bootstrap-based PIPs exist to visualize)
- The goal is to visualize the G-Z-Y cluster structure itself rather than variable-inclusion
  uncertainty — use the Sankey diagram via plot_lucid() instead
- The analysis does not involve latent clusters integrating exposures, omics data,
  and an outcome (e.g., a plain univariate exposure-outcome regression)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LUCIDus
  role: Provides summary_lucid() to extract bootstrap-based posterior inclusion probabilities
    from a fitted LUCID model for plotting
  repo: https://github.com/USCbiostats/LUCIDus
- name: LUCIDus3
  role: Current CRAN version (3.1.0) with bootstrap inference (boot_lucid()) and model
    summaries for early, parallel, and serial LUCID models
  repo: https://github.com/USCbiostats/LUCIDus3
- name: R
  role: Implementation and plotting environment for the LUCIDus package and the PIP
    histogram
  repo: https://github.com/USCbiostats/LUCIDus
- name: mclust
  role: Default initialization method (mclust-based regression) used when fitting
    the LUCID model whose PIPs are visualized
  repo: https://github.com/USCbiostats/LUCIDus
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:903790cd36c105df0c1d8e48686911e496894aef7de020ead46c9d7776bba111
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - LUCIDus
  - LUCIDus3
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Posterior inclusion probability visualization

## Summary

Visualize bootstrap-based posterior inclusion probabilities (PIPs) of exposures and omic features from a fitted LUCID model, typically as a PIP histogram annotated with missingness ratios, to judge which variables drive the latent clusters.

## When to use

Use when a fitted LUCIDus model (e.g., HCB exposure, incomplete proteomics, childhood BMI z-score) yields bootstrap-based posterior inclusion probabilities and you need to visualize and interpret variable inclusion uncertainty.

## When NOT to use

- The model has not been fitted with LUCIDus (no bootstrap-based PIPs exist to visualize)
- The goal is to visualize the G-Z-Y cluster structure itself rather than variable-inclusion uncertainty — use the Sankey diagram via plot_lucid() instead
- The analysis does not involve latent clusters integrating exposures, omics data, and an outcome (e.g., a plain univariate exposure-outcome regression)

## Inputs

- Fitted LUCID model object from lucid() or estimate_lucid() (R, LUCIDus / LUCIDus3)
- Exposure matrix G (prenatal hexachlorobenzene, optionally as quartiles)
- Omics matrix Z (proteomics with list-wise missingness from 1301 children)
- Outcome Y (childhood BMI z-score)
- Bootstrap resampling results underlying the posterior inclusion probabilities

## Outputs

- PIP histogram figure (e.g., pip_histogram.png) annotated with HCB-quartile missingness ratios (2.56%-37.50%)
- Summary table of PIPs and parameter estimates (e.g., lucid_model_summary.csv) with cluster-specific mean BMI z-scores (-1.78, -1.52)

## How to apply

After fitting the LUCID model with lucid() (or estimate_lucid() in LUCIDus3) on the exposure matrix G (e.g., prenatal hexachlorobenzene quartiles), omics matrix Z (proteomics with list-wise missingness), and outcome Y (childhood BMI z-score), call summary_lucid() to obtain the bootstrap-based posterior inclusion probabilities alongside parameter estimates such as cluster-specific mean BMI z-scores. Plot the PIPs as a histogram (e.g., with R's hist() or ggplot2), annotating each PIP bin with the corresponding HCB-quartile missingness ratios (which ranged from 2.56% to 37.50% in the Exposome Data Challenge application) so readers can gauge how imputation may affect inclusion certainty. Save the figure (e.g., pip_histogram.png) together with the numeric summary (e.g., lucid_model_summary.csv) for reproducibility. Interpret high-PIP exposures/omics as variables that reliably distinguish the latent clusters, and cross-check against the model's automatic BIC-based K selection (K=2) and the Sankey diagram from plot_lucid() showing the G-Z-Y interplay.

## Related tools

- **LUCIDus** (Provides summary_lucid() to extract bootstrap-based posterior inclusion probabilities from a fitted LUCID model for plotting) — https://github.com/USCbiostats/LUCIDus
- **LUCIDus3** (Current CRAN version (3.1.0) with bootstrap inference (boot_lucid()) and model summaries for early, parallel, and serial LUCID models) — https://github.com/USCbiostats/LUCIDus3
- **R** (Implementation and plotting environment for the LUCIDus package and the PIP histogram) — https://github.com/USCbiostats/LUCIDus
- **mclust** (Default initialization method (mclust-based regression) used when fitting the LUCID model whose PIPs are visualized) — https://github.com/USCbiostats/LUCIDus

## Examples

```
fit <- lucid(G = G, Z = Z, Y = Y, K = 2); s <- summary_lucid(fit); hist(s$pip)
```

## Evaluation signals

- All plotted PIP values lie in [0, 1] and correspond 1:1 to variables in G and Z of the fitted model
- The histogram is annotated with the correct missingness ratios per HCB quartile (2.56%-37.50%), matching the input proteomics missingness pattern
- PIP summary values agree with the summary_lucid() output saved to lucid_model_summary.csv (cluster-specific mean BMI z-scores -1.78 and -1.52)
- Bootstrap CIs/PIPs are reproducible with a fixed seed (e.g., seed = 1008 as used in LUCIDus3 examples)
- High-PIP variables are consistent with the model's BIC-selected K=2 clustering and the exposure-to-cluster odds ratios (1.2, 1.71, 3.19)

## Limitations

- PIPs reflect bootstrap resampling of a single fitted model; list-wise missing proteomics (handled by the integrated imputation framework) can affect inclusion certainty, so missingness ratios should be annotated alongside PIPs
- PIPs pertain to the LUCID model family (early/parallel/serial as supported by LUCIDus3); they are not defined for external clustering or regression models
- The paper's discussion section is absent ('No changelog found.'), so no explicit published caveats on PIP interpretation are available beyond the methods described

## Evidence

- [intro] summary_lucid() summarizes the fitted LUCID model by producing summary tables of parameter estimation and interpretation.: "`summary_lucid()` summarizes the fitted LUCID model by producing summary tables of parameter estimation and interpretation."
- [intro] Bootstrap-based inference underlies the PIPs used in the visualization.: "statistical inference can be accomplished for LUCID by constructing confidence intervals (CIs) based on bootstrap resampling."
- [readme] The LUCIDus package features bootstrap inference used to derive the inclusion probabilities.: "**LUCIDus** features variable selection, incorporating missingness in omics data, visualization of LUCID model via Sankey diagram, bootstrap"
- [readme] Bootstrap inference is a first-class feature of the maintained LUCIDus3 package.: [evidence span withheld — anchor status: external_doc]
- [abstract] The integrated imputation approach justifies annotating PIPs with missingness ratios.: "Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values."
