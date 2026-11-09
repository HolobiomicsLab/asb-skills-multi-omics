---
name: list-wise-missing-omics-imputation
description: Use when an exposure/omics/outcome (G, Z, Y) matrix intended for LUCID
  integrative clustering has list-wise missing rows in the omics block (e.g., only
  a subset of the 1301 HELIX/Exposome Data Challenge children have measured proteomics)
  and you need unbiased cluster estimates.
when_to_use_negative:
- The omics matrix Z is complete (no list-wise missing rows) — the complete-data LUCID
  workflow needs no integrated imputation
- The analysis goal is a standard one-exposure-one-disease regression rather than
  joint latent-cluster estimation across G, Z, and Y
- You plan to rely on impute-mice for high list-wise missing ratios (>0.5), where
  this paper shows drastic bias toward 0
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LUCIDus
  role: Implements the LUCID model with integrated imputation for missing omics data,
    including lucid() fitting, BIC-based K selection, bootstrap inference, and Sankey
    visualization
  repo: https://github.com/USCbiostats/LUCIDus
- name: LUCIDus3
  role: Actively maintained extension (CRAN LUCIDus 3.1.0) supplying missing-data
    diagnostics and imputation helpers such as analyze_missing_pattern(), check_na(),
    safe_impute(), fill_data(), and check_imputation_quality()
  repo: https://github.com/USCbiostats/LUCIDus3
- name: mclust
  role: 'Default initialization method: mclust-based regression initializes LUCID
    parameters before EM estimation on the incomplete omics'
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:9abb7b19161b1ab987ce9f42bf9090b1017f83d76841c1edf7806687277a1d0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - LUCIDus
  - LUCIDus3
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# list-wise-missing-omics-imputation

## Summary

Impute list-wise (block-wise) missing omics data inside the LUCID integrated clustering framework rather than preprocessing with external imputation, using mclust-based regression initialization and EM estimation so exposure-effect estimates remain unbiased and clustering AUC stays stable as missing ratios rise from 0.1 to 0.8.

## When to use

Use when an exposure/omics/outcome (G, Z, Y) matrix intended for LUCID integrative clustering has list-wise missing rows in the omics block (e.g., only a subset of the 1301 HELIX/Exposome Data Challenge children have measured proteomics) and you need unbiased cluster estimates.

## When NOT to use

- The omics matrix Z is complete (no list-wise missing rows) — the complete-data LUCID workflow needs no integrated imputation
- The analysis goal is a standard one-exposure-one-disease regression rather than joint latent-cluster estimation across G, Z, and Y
- You plan to rely on impute-mice for high list-wise missing ratios (>0.5), where this paper shows drastic bias toward 0

## Inputs

- Exposure matrix G (e.g., prenatal hexachlorobenzene levels, optionally by quartile)
- Omics matrix Z with list-wise missing rows (e.g., proteomics with 2.56%–37.50% missingness ratios)
- Outcome Y (e.g., childhood BMI z-score, normal family)

## Outputs

- Fitted LUCID model object with integrated imputation and EM parameter estimates
- BIC-selected number of latent clusters K (e.g., K=2)
- Bootstrap posterior inclusion probabilities and confidence intervals
- Cluster-specific outcome summaries (e.g., mean BMI z-scores -1.78, -1.52) and exposure-to-cluster odds ratios (1.2, 1.71, 3.19)
- Sankey diagram (plot_lucid()) and PIP histogram of the fitted model

## How to apply

First diagnose the missing pattern (list-wise vs. sporadic) with LUCIDus3 helpers such as analyze_missing_pattern()/check_na(), then fit the LUCID model with lucid() or estimate_lucid() using the integrated imputation option ('L' method) rather than complete-case analysis or external imputation like impute-mice, impute-mclust, or impute-EM. Initialize parameters with the default mclust-and-regression initializer, let the EM algorithm jointly estimate latent clusters while imputing the incomplete omics, and let lucid() auto-select the cluster number K via BIC (K=2 in the HELIX proteomics/BMI application). Evaluate the fit by bootstrap-based posterior inclusion probabilities and CIs, and summarize with summary_lucid() and plot_lucid() (Sankey of G–Z–Y). Rationale: simulations across list-wise missing ratios of 0.1–0.8 (8000 training/2000 validation, 300 replications, 10 exposures, 4 omics, K=2) show the integrated approach stays centered on true values with the smallest SDs and highest, most stable median AUC, whereas impute-mice becomes drastically biased toward 0 above a 0.5 missing ratio.

## Related tools

- **LUCIDus** (Implements the LUCID model with integrated imputation for missing omics data, including lucid() fitting, BIC-based K selection, bootstrap inference, and Sankey visualization) — https://github.com/USCbiostats/LUCIDus
- **LUCIDus3** (Actively maintained extension (CRAN LUCIDus 3.1.0) supplying missing-data diagnostics and imputation helpers such as analyze_missing_pattern(), check_na(), safe_impute(), fill_data(), and check_imputation_quality()) — https://github.com/USCbiostats/LUCIDus3
- **mclust** (Default initialization method: mclust-based regression initializes LUCID parameters before EM estimation on the incomplete omics)

## Examples

```
library(LUCIDus); fit <- lucid(G = hcb_quartiles, Z = proteomics_with_missing, Y = bmi_z, K = 2, seed = 1008)
```

## Evaluation signals

- Exposure-effect parameter estimates from the integrated imputation ('L') fit remain centered on the true value with the smallest standard deviations in replication-based bias/SD tables (e.g., the 300-replication, 0.1–0.8 missing-ratio simulation)
- Median cluster/outcome prediction AUC on a held-out validation set (e.g., 2000 samples) is stable and highest versus complete-case, impute-mclust, impute-mice, and impute-EM competitors
- BIC-based automatic model selection recovers the true cluster number (K=2 in both simulation and the 1301-child HELIX proteomics application)
- Bootstrap posterior inclusion probabilities are computed for exposure and omic effects, and summaries via summary_lucid() reproduce expected cluster means (e.g., BMI z-scores -1.78 and -1.52)
- Missingness diagnosis precedes fitting: the pattern is confirmed as list-wise (subset of observations with measured omics), with per-variable missingness ratios recorded (e.g., 2.56%–37.50%)

## Limitations

- impute-mice becomes drastically biased toward 0, especially at list-wise missing ratios above 0.5, so external imputation pipelines are unreliable benchmarks at high missingness
- The integrated imputation framework targets list-wise missingness and only 'can also handle sporadic missingness when necessary', so predominantly sporadic patterns are a secondary use case
- The discussion section content is absent from the extracted article ('No changelog found.'), so no extractable limitations or reproducibility-gap notes are available beyond the simulation scope

## Evidence

- [abstract] extension to list-wise missingness within integrated imputation: "we extend the model to incorporate list-wise missingness within an integrated imputation framework, which can also handle sporadic missingness when necessary."
- [intro] two missing patterns considered: "We consider 2 missing patterns in omics data: (1) list-wise missing pattern that only a subset of observations have measured omics data"
- [intro] mclust initialization is the default: "The first method (also the default method) uses `mclust` and regression to initialize parameters."
- [readme] imputation helpers in LUCIDus3 README: [evidence span withheld — anchor status: external_doc]
- [readme] imputation API in LUCIDus3 README: [evidence span withheld — anchor status: external_doc]
