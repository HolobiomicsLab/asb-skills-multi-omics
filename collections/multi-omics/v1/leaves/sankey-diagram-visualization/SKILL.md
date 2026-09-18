---
name: sankey-diagram-visualization
description: Use when you have a fitted LUCID model (from lucid()/estimate_lucid())
  built on an exposure matrix G, omics matrix Z, and outcome Y, and you need to communicate
  the G-Z-Y interplay and cluster flows visually.
when_to_use_negative:
- The fitted model is a 'parallel' or 'serial' LUCID model — per the LUCIDus3 README,
  Sankey-style plots are supported for early models only
- No LUCID model has been fitted yet and no latent cluster structure exists to visualize
  — build the model with lucid()/estimate_lucid() first
- The goal is statistical inference (confidence intervals) rather than visualization
  — use boot_lucid()/bootstrap inference instead
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: LUCIDus
  role: Provides plot_lucid() to create the Sankey diagram of the fitted LUCID model's
    G-Z-Y structure
  repo: https://github.com/USCbiostats/LUCIDus
- name: LUCIDus3
  role: Current CRAN package (LUCIDus 3.1.0) whose S3 plot() method visualizes fitted
    early models with Sankey-style plots
  repo: https://github.com/USCbiostats/LUCIDus3
- name: R
  role: Language environment in which the LUCIDus visualization functions are run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:05a1345792df584b4cfe18bffa384d458edd8608c0664102de81f97bee22e25c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - LUCIDus
  - LUCIDus3
  - R
  - mclust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2857142857142857
n_claims: 7
---

# sankey-diagram-visualization

## Summary

Visualize a fitted LUCID integrative-clustering model as a Sankey diagram depicting the interplay among exposures (G), omics data (Z), and outcome (Y), including cluster-specific flows such as exposure-quartile-to-latent-cluster odds ratios. Implemented in the R package LUCIDus via plot_lucid() (or plot() for early models in LUCIDus 3.x).

## When to use

Use when you have a fitted LUCID model (from lucid()/estimate_lucid()) built on an exposure matrix G, omics matrix Z, and outcome Y, and you need to communicate the G-Z-Y interplay and cluster flows visually.

## When NOT to use

- The fitted model is a 'parallel' or 'serial' LUCID model — per the LUCIDus3 README, Sankey-style plots are supported for early models only
- No LUCID model has been fitted yet and no latent cluster structure exists to visualize — build the model with lucid()/estimate_lucid() first
- The goal is statistical inference (confidence intervals) rather than visualization — use boot_lucid()/bootstrap inference instead

## Inputs

- Fitted LUCID model object returned by lucid() or estimate_lucid() (early integration)
- Exposure matrix G (e.g., prenatal hexachlorobenzene quartiles)
- Omics matrix Z (e.g., proteomics with list-wise missingness handled by integrated imputation)
- Outcome Y (e.g., childhood BMI z-score)
- Selected number of latent clusters K (e.g., K=2 from BIC-based model selection)

## Outputs

- Sankey diagram image (e.g., sankey_diagram.png) showing the interplay among G, Z, and Y
- Annotated flows such as exposure-quartile-to-cluster odds ratios (1.2, 1.71, 3.19) linked to cluster-specific mean BMI z-scores (-1.78, -1.52)

## How to apply

After fitting the LUCID model with lucid() — e.g., the HCB-BMI analysis on the ISGlobal/ATHLETE ExposomeDataChallenge2021 mother-child data with 1301 children and list-wise missing proteomics, imputed via the integrated imputation framework and K=2 selected by BIC — call plot_lucid() on the fitted object to render a Sankey diagram linking exposures to latent clusters to outcome. In the reference analysis the diagram annotates the exposure-to-cluster odds ratios for increasing prenatal hexachlorobenzene quartiles (1.2, 1.71, 3.19) flowing into the cluster that is associated with higher childhood BMI, and the output is saved as sankey_diagram.png alongside summary_lucid() tables (lucid_model_summary.csv, PIP histogram) to jointly convey G-Z-Y structure. In LUCIDus 3.1.0 the equivalent call for early-integration models is the S3 plot() method. Verify the diagram by checking that node counts match K (here 2 clusters), flows correspond to reported ORs, and the visualized topology mirrors summary_lucid() output.

## Related tools

- **LUCIDus** (Provides plot_lucid() to create the Sankey diagram of the fitted LUCID model's G-Z-Y structure) — https://github.com/USCbiostats/LUCIDus
- **LUCIDus3** (Current CRAN package (LUCIDus 3.1.0) whose S3 plot() method visualizes fitted early models with Sankey-style plots) — https://github.com/USCbiostats/LUCIDus3
- **R** (Language environment in which the LUCIDus visualization functions are run)

## Examples

```
library(LUCIDus); plot_lucid(fit)  # Sankey diagram of G-Z-Y interplay; in LUCIDus 3.x: plot(fit_early)
```

## Evaluation signals

- The Sankey diagram contains exactly K cluster nodes matching the BIC-selected K (K=2 in the HCB-BMI analysis)
- Exposure-to-cluster flows reproduce the reported odds ratios (1.2, 1.71, 3.19 for HCB quartiles 2-4)
- Cluster nodes align with summary_lucid() cluster-specific mean BMI z-scores (-1.78, -1.52), with the higher-BMI cluster correctly linked
- The rendered diagram file (sankey_diagram.png) matches the topology in summary_lucid() output (lucid_model_summary.csv) — same G, Z, and Y components
- Visualization is generated on the model after imputation of list-wise missing proteomics, so no missingness-related nodes or NaN flows appear

## Limitations

- Sankey-style plots in LUCIDus 3.1.0 are documented for early-integration models; parallel and serial model topologies are not covered by this visualization
- The diagram is descriptive: it displays the G-Z-Y interplay and ORs but does not itself provide uncertainty quantification — bootstrap-based inference (CIs) is a separate step
- The original LUCIDus2 repository is retained only 'for reproducibility of the original published method'; new projects are directed to LUCIDus3

## Evidence

- [intro] Visualization is performed via the `plot_lucid()` function to create a Sankey diagram showing the interplay among the three components (`G`, `Z` and `Y`).: "Visualization is performed via the `plot_lucid()` function to create a Sankey diagram showing the interplay among the three components (`G`, `Z` and `Y`)."
- [readme] **LUCIDus** features variable selection, incorporating missingness in omics data, visualization of LUCID model via Sankey diagram, bootstrap: "**LUCIDus** features variable selection, incorporating missingness in omics data, visualization of LUCID model via Sankey diagram, bootstrap"
- [readme] Visualize fitted early models | `plot()`: [evidence span withheld — anchor status: external_doc]
- [readme] S3 summaries for all supported model types and **Sankey-style plots** for early models.: [evidence span withheld — anchor status: not_found]
- [intro] `summary_lucid()` summarizes the fitted LUCID model by producing summary tables of parameter estimation and interpretation.: "`summary_lucid()` summarizes the fitted LUCID model by producing summary tables of parameter estimation and interpretation."
