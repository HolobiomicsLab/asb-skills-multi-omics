---
name: sensitivity-analysis-vs-imputation-benchmark
description: Use when a LUCID model was fit with integrated imputation on incomplete
  omics data and you need a sensitivity check comparing complete-case parameter estimates
  side by side with the imputed-model results.
when_to_use_negative:
- The omics data are complete (no list-wise missingness) — the complete-case and imputation
  analyses coincide, so the benchmark adds no information
- Missingness is sporadic rather than list-wise; the paper's imputation framework
  handles sporadic patterns that list-wise deletion would discard
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0091
tools:
- name: LUCIDus
  role: Provides lucid() to fit the complete-case LUCID model and summary_lucid()
    to extract cluster-specific parameter estimates for comparison
  repo: https://github.com/USCbiostats/LUCIDus
- name: mclust
  role: 'Default initialization method: mclust and regression initialize LUCID parameters
    before EM estimation'
- name: R
  role: Environment for loading the Exposome Data Challenge data, subsetting to complete
    cases, and writing the comparison CSV
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:d2768b489b69cb32cac41a7b11842877472c24fc0a6f61b3105e873dd5dec490
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - LUCIDus
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Sensitivity analysis vs imputation benchmark

## Summary

Benchmarks a complete-case (list-wise deletion) LUCID analysis against the integrated-imputation reference model on the same HCB–proteomics–BMI data, so practitioners can quantify how much the imputation framework changes cluster-specific estimates such as the BMI z-score means (cluster 1 = -1.78 vs cluster 2 = -1.52).

## When to use

Use when a LUCID model was fit with integrated imputation on incomplete omics data and you need a sensitivity check comparing complete-case parameter estimates side by side with the imputed-model results.

## When NOT to use

- The omics data are complete (no list-wise missingness) — the complete-case and imputation analyses coincide, so the benchmark adds no information
- Missingness is sporadic rather than list-wise; the paper's imputation framework handles sporadic patterns that list-wise deletion would discard

## Inputs

- ISGlobal/ATHLETE Exposome Data Challenge 2021 dataset (maternal hexachlorobenzene exposure, proteomics from 1301 children, childhood BMI z-score)
- Reference integrated-imputation Model 1 results from Supplementary section C
- Fitted LUCID model object from lucid() (complete-case fit)

## Outputs

- complete_case_comparison.csv with complete-case vs integrated-imputation parameter estimates side by side
- Cluster-specific exposure effects, omics profiles, and outcome distributions from summary_lucid()
- Selected optimal number of clusters K (via BIC)

## How to apply

Restrict the ISGlobal/ATHLETE Exposome Data Challenge 2021 dataset (maternal hexachlorobenzene, proteomics from 1301 children, childhood BMI) to children with fully measured proteomics profiles to form the complete-case set. Fit the LUCID model with lucid() in LUCIDus (G = HCB exposure, Z = proteomics, Y = BMI), using mclust-based regression for parameter initialization and selecting the optimal number of clusters K via BIC. Summarize the fitted complete-case model with summary_lucid() to extract cluster-specific exposure effects, omics profiles, and outcome distributions. Tabulate the complete-case estimates side by side with the Model 1 integrated-imputation results from Supplementary section C and write the comparison to complete_case_comparison.csv. Judge success by checking that the complete-case model recovers the same number of clusters and qualitatively similar ordering (cluster 2 higher mean scaled BMI z-score than cluster 1) as the imputation reference.

## Related tools

- **LUCIDus** (Provides lucid() to fit the complete-case LUCID model and summary_lucid() to extract cluster-specific parameter estimates for comparison) — https://github.com/USCbiostats/LUCIDus
- **mclust** (Default initialization method: mclust and regression initialize LUCID parameters before EM estimation)
- **R** (Environment for loading the Exposome Data Challenge data, subsetting to complete cases, and writing the comparison CSV)

## Examples

```
fit_cc <- lucid(G = hcb, Z = proteomics_complete_cases, Y = bmi_z, K = 1:5, seed = 1008); summary_lucid(fit_cc)
```

## Evaluation signals

- complete_case_comparison.csv exists and contains paired complete-case and Model 1 integrated-imputation estimates for the same parameters
- Both analyses select the same optimal number of clusters K (expected 2) via BIC
- Cluster ordering is preserved: complete-case cluster 2 mean scaled BMI z-score is higher than cluster 1, matching the imputation reference (-1.52 vs -1.78)
- Complete-case set equals the subset of children with fully measured proteomics profiles (no missing values in Z)
- summary_lucid() output covers exposure effects, omics profiles, and outcome distributions for every cluster

## Limitations

- Complete-case analysis discards children with any missing proteomics measurements, reducing sample size relative to the 1301-child imputation analysis
- The paper's simulation results indicate the integrated imputation approach produces more consistent, less biased estimates than alternatives, so complete-case results are expected to deviate and serve only as a sensitivity check
- Discussion section of the article is absent ('No changelog found.'), so no explicit reproducibility caveats or limitation text could be extracted

## Evidence

- [other] Complete-case set definition: [evidence span withheld — anchor status: not_found]
- [intro] Model fitting and selection: "The function `lucid()` is the primary function in the package, which fits a LUCID model based on an exposure matrix (argument `G`), omics data (argument `Z`), outcome data (argument `Y`)"
- [intro] Initialization method: "The first method (also the default method) uses `mclust` and regression to initialize parameters."
- [intro] Summarization function: "`summary_lucid()` summarizes the fitted LUCID model by producing summary tables of parameter estimation and interpretation."
- [abstract] Application dataset: "to explore the association between maternal exposure to hexachlorobenzene and childhood body mass index by integrating incomplete proteomics data from 1301 children"
- [abstract] Imputation superiority rationale: "Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values."
