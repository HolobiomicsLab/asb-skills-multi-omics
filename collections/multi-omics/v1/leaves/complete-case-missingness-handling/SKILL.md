---
name: complete-case-missingness-handling
description: Use when reproducing the complete-case baseline (Supplementary section
  C comparison) for the HCB–BMI LUCID analysis, i.e. when the omics matrix has list-wise
  missingness and you need a fully-measured subset for benchmarking.
when_to_use_negative:
- When the goal is full-sample inference on all 1301 children — use the integrated-imputation
  LUCID framework instead of discarding incompletely measured observations
- When missingness is sporadic (scattered cell-wise) rather than list-wise, where
  dropping entire rows wastes far more data than necessary
- When no omics values are missing, since the complete-case restriction becomes a
  no-op and the imputation-vs-complete-case comparison is uninformative
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3337
tools:
- name: LUCIDus
  role: Fits the LUCID model via lucid() on the complete-case subset (G = HCB, Z =
    proteomics, Y = BMI) and summarizes it via summary_lucid()
  repo: https://github.com/USCbiostats/LUCIDus
- name: mclust
  role: Provides the mclust and regression based initialization of LUCID model parameters,
    the default method
- name: R
  role: Language environment in which the complete-case filtering, LUCID fitting,
    and comparison-table export are scripted
  repo: https://github.com/USCbiostats/LUCIDus
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:6f4cf58edbb35b04b306cddd039cfe1c3ba3a63cca33f00ac83685ab33f3282f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - LUCIDus
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3337
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# complete-case-missingness-handling

## Summary

Form a complete-case analysis set by restricting the ISGlobal/ATHLETE Exposome Data Challenge 2021 data to children with fully measured proteomics profiles, then fit the LUCID model (G = maternal hexachlorobenzene, Z = proteomics, Y = childhood BMI z-score) on this subset as a baseline against which the integrated-imputation approach is compared.

## When to use

Use when reproducing the complete-case baseline (Supplementary section C comparison) for the HCB–BMI LUCID analysis, i.e. when the omics matrix has list-wise missingness and you need a fully-measured subset for benchmarking.

## When NOT to use

- When the goal is full-sample inference on all 1301 children — use the integrated-imputation LUCID framework instead of discarding incompletely measured observations
- When missingness is sporadic (scattered cell-wise) rather than list-wise, where dropping entire rows wastes far more data than necessary
- When no omics values are missing, since the complete-case restriction becomes a no-op and the imputation-vs-complete-case comparison is uninformative

## Inputs

- ISGlobal/ATHLETE Exposome Data Challenge 2021 dataset: maternal hexachlorobenzene (HCB) exposure, proteomics from 1301 children, childhood BMI z-score
- List-wise missingness indicator over the proteomics matrix (children with fully measured vs. partially measured profiles)
- Supplementary section C Model 1 integrated-imputation results for side-by-side comparison

## Outputs

- Complete-case analysis set (children with fully measured proteomics profiles)
- Fitted LUCID model summary (cluster-specific exposure effects, omics profiles, outcome distributions) from summary_lucid()
- complete_case_comparison.csv: complete-case parameter estimates tabulated against Model 1 integrated-imputation results

## How to apply

Load the Exposome Data Challenge 2021 dataset (maternal hexachlorobenzene exposure, proteomics from 1301 children, childhood BMI) and restrict it to complete cases — children with fully measured proteomics profiles — discarding rows with any missing omics values. On this subset, fit the LUCID model with the lucid() function in LUCIDus (G = HCB exposure, Z = proteomics, Y = BMI), using mclust and regression for parameter initialization and selecting the optimal number of clusters K via BIC. Summarize cluster-specific exposure effects, omics profiles, and outcome distributions with summary_lucid(). Rationale: complete-case analysis avoids imputation entirely but discards observations, so it serves as the reference against which the integrated-imputation LUCID model (which the simulation studies show produces consistent and less biased estimates) is judged; in the HCB–BMI application cluster 2 showed a higher mean scaled BMI z-score than cluster 1 (μBMI Cluster 1 = -1.78; μBMI Cluster 2 = -1.52). Finally, tabulate the complete-case estimates side by side with the Model 1 integrated-imputation results and export the comparison table as complete_case_comparison.csv.

## Related tools

- **LUCIDus** (Fits the LUCID model via lucid() on the complete-case subset (G = HCB, Z = proteomics, Y = BMI) and summarizes it via summary_lucid()) — https://github.com/USCbiostats/LUCIDus
- **mclust** (Provides the mclust and regression based initialization of LUCID model parameters, the default method)
- **R** (Language environment in which the complete-case filtering, LUCID fitting, and comparison-table export are scripted) — https://github.com/USCbiostats/LUCIDus

## Examples

```
fit <- lucid(G = HCB_cc, Z = proteomics_cc, Y = BMI_cc); summary_lucid(fit)
```

## Evaluation signals

- Every child retained in the complete-case set has a fully measured proteomics profile (zero NA values across all omics columns)
- Optimal number of clusters K is selected by BIC during lucid() model selection
- Cluster-specific mean scaled BMI z-scores are recoverable and comparable to the reference ordering (cluster 2 > cluster 1: -1.52 vs -1.78)
- complete_case_comparison.csv exists and contains side-by-side complete-case vs. Model 1 integrated-imputation parameter estimates
- Summary output from summary_lucid() reports cluster-specific exposure effects, omics profiles, and outcome distributions for the fitted model

## Limitations

- Complete-case analysis discards children with list-wise missing proteomics data, reducing sample size relative to the 1301-child integrated-imputation analysis
- The integrated imputation approach is expected to outperform it: simulations show the integrated imputation approach produces consistent and less biased estimates closely reflecting true underlying values
- The article does not extractable discussion content on limitations ('No changelog found.'), so edge-case guidance beyond the two missingness patterns is not documented

## Evidence

- [abstract] list-wise missingness framing: "To address this, we extend the model to incorporate list-wise missingness within an integrated imputation framework, which can also handle sporadic missingness when necessary."
- [abstract] simulation benchmark favoring imputation: "Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values."
- [intro] two missing patterns considered: "(1) list-wise missing pattern that only a subset of observations have measured omics data"
- [other] complete-case filtering step: [evidence span withheld — anchor status: not_found]
- [readme] missingness tooling in current package: [evidence span withheld — anchor status: external_doc]
