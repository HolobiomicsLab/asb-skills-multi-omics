---
name: dropout-noise-injection
description: Use when benchmarking ST integration methods (e.g. stImage vs MUSE) on
  synthetic data that must include controlled transcriptomic missingness, or when
  reconstructing simulated ST datasets with distinct/ambiguous spatial patterns across
  4, 6, 8, or 10 clusters.
when_to_use_negative:
- Input is real experimental ST data (e.g. HER2 breast cancer ST or Visium MOB) —
  dropout injection is only for synthetic benchmark data, not for perturbing observed
  measurements
- The goal is denoising or imputation of an existing dataset rather than controlled
  benchmarking of integration strategies
- The simulation does not include a paired image-feature modality, so the MUSE-style
  two-modality design being evaluated cannot be tested
tools:
- name: stImage
  role: R package environment in which the simulated data are generated and whose
    54 integrative strategies are benchmarked against the dropout-masked data
  repo: https://github.com/YuWang-VUMC/stImage
- name: tidyverse
  role: Loaded in R to support the simulation and masking workflow
- name: MUSE
  role: 'Reference simulation design: expression drawn from a multivariable normal
    distribution with paired image features follows the MUSE two-modality setup'
- name: R
  role: Implementation language; masked data are saved and validated as simulated_st_data.rds
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:ce3d57e34d98644e2c51446e722ecbbab145b15f5e21f07f16ccb33a7d60e214
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuWang-VUMC/stImage
  tools:
  - MUSE
  - R
  - stImage
  - tidyverse
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# dropout-noise-injection

## Summary

Simulates dropout noise in spatial transcriptomics benchmark data by applying controlled masking (60%, 70%, 80%, 90% dropout rates) to simulated spot-level gene expression matrices generated under the MUSE two-modality design, enabling stImage's integrative strategies to be stress-tested under varying missingness.

## When to use

Use when benchmarking ST integration methods (e.g. stImage vs MUSE) on synthetic data that must include controlled transcriptomic missingness, or when reconstructing simulated ST datasets with distinct/ambiguous spatial patterns across 4, 6, 8, or 10 clusters.

## When NOT to use

- Input is real experimental ST data (e.g. HER2 breast cancer ST or Visium MOB) — dropout injection is only for synthetic benchmark data, not for perturbing observed measurements
- The goal is denoising or imputation of an existing dataset rather than controlled benchmarking of integration strategies
- The simulation does not include a paired image-feature modality, so the MUSE-style two-modality design being evaluated cannot be tested

## Inputs

- Simulated spot-level gene expression matrix (multivariable normal, MUSE-style, k = 4/6/8/10 clusters)
- Simulated spot-level image feature profiles matched to the spatial patterns
- Spatial coordinates of simulated spots
- Ground-truth cluster labels per spot
- Target dropout rate (one of 60%, 70%, 80%, 90%)

## Outputs

- Dropout-masked simulated expression matrix at each specified rate
- simulated_st_data.rds containing expression, image features, spatial coordinates, and ground-truth labels for every k and dropout-rate combination

## How to apply

In the stImage R environment (with tidyverse), first generate spot-level gene expression from a multivariable normal distribution following the MUSE simulation setup, embedding either distinct or ambiguous spatial patterns across spots, with k = 4, 6, 8, or 10 clusters. Generate matching spot-level image feature profiles consistent with the assigned spatial patterns to mirror MUSE's two-modality design. Then apply dropout masking to the simulated expression matrix at each specified rate (60%, 70%, 80%, 90%), masking entries at random so the effective observed count matches the target dropout level. Finally, assemble the masked expression, image features, spatial coordinates, and ground-truth cluster labels into a single object (e.g. simulated_st_data.rds) for each k and dropout-rate combination. Rationale: dropout injection exposes how modality-integration versus spatially-aware strategies in stImage depend on expression completeness, since existing ST methods are 'highly dependent on assumptions about transcriptomics, spatial, and image patterns'.

## Related tools

- **stImage** (R package environment in which the simulated data are generated and whose 54 integrative strategies are benchmarked against the dropout-masked data) — https://github.com/YuWang-VUMC/stImage
- **tidyverse** (Loaded in R to support the simulation and masking workflow)
- **MUSE** (Reference simulation design: expression drawn from a multivariable normal distribution with paired image features follows the MUSE two-modality setup)
- **R** (Implementation language; masked data are saved and validated as simulated_st_data.rds)

## Examples

```
set.seed(42); mask <- matrix(runif(nrow(expr)*ncol(expr)) < 0.8, nrow(expr)); expr_masked <- expr; expr_masked[mask] <- 0; saveRDS(list(expr=expr_masked, img=img_feat, coords=coords, labels=truth), "simulated_st_data.rds")
```

## Evaluation signals

- The saved simulated_st_data.rds loads in R and contains all four components (expression, image features, spatial coordinates, ground-truth labels) for every k and dropout-rate combination
- The number of distinct ground-truth labels in each simulated dataset equals k (4, 6, 8, or 10)
- The observed fraction of non-zero expression entries after masking matches the target dropout rate (60%, 70%, 80%, or 90%) within sampling tolerance
- Simulated image feature profiles are consistent with the assigned spatial patterns (distinct vs ambiguous), mirroring the MUSE two-modality design
- Downstream benchmark runs reproduce the paper's contrast between distinct and ambiguous pattern settings

## Limitations

- Only four discrete dropout rates (60/70/80/90%) are defined in the main simulation grid; intermediate rates are not covered
- The simulation relies on multivariable-normal expression assumptions, which may not capture real transcriptomic distributions
- Because no single integration approach performed well on every dataset tested, dropout-robustness results from these simulations should not be generalized to arbitrary real tissues without the diagnostic-graph guidance stImage provides

## Evidence

- [methods] Generate spot-level gene expression from a multivariable normal distribution following the MUSE simulation setup, embedding both distinct and ambiguous spatial patterns across spots.: [evidence span withheld — anchor status: not_found]
- [methods] Apply dropout masking at each specified rate (60/70/80/90%) to the simulated expression matrices.: [evidence span withheld — anchor status: not_found]
- [results] Simulated ST datasets were generated exhibiting either distinct or ambiguous spatial patterns, with varying numbers of clusters (10 clusters in the main setting; 4, 6, and 8 clusters in additional settings).: [evidence span withheld — anchor status: not_found]
- [intro] MUSE learns joint latent features by integrating morphological image features and transcriptomic data.: "MUSE [4], for example, learns joint latent features by integrating morphological image features and transcriptomic data"
- [readme] stImage is an R package for integrated analysis of spatial transcriptomics and corresponding modality images.: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images"
