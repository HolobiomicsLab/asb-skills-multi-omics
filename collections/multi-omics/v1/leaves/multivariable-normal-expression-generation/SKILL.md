---
name: multivariable-normal-expression-generation
description: Use when benchmarking ST integration/clustering methods (e.g. stImage
  vs MUSE-style baselines) requires synthetic two-modality data with known ground-truth
  labels, controlled cluster counts, and controlled expression dropout.
when_to_use_negative:
- The target dataset is real tissue data with no simulation grid needed — use stImage's
  PreProcessing/MultiModalIntegration workflow on real data instead.
- The goal is to evaluate methods on a single fixed cluster count without dropout
  stress-testing — the grid of 4/6/8/10 clusters and 60–90% dropout is unnecessary
  overhead.
- Only image modality or only spatial coordinates are of interest — this skill specifically
  generates paired expression + image feature two-modality data.
edam_operation: http://edamontology.org/operation_0244
tools:
- name: stImage
  role: R package environment in which the simulated expression, image features, coordinates,
    and labels are generated and assembled
  repo: https://github.com/YuWang-VUMC/stImage
- name: tidyverse
  role: Loaded in R for data manipulation while building the simulation grid and assembling
    the .rds output
- name: MUSE
  role: Reference simulation setup whose two-modality (expression + image features)
    design the simulation mirrors
- name: R
  role: Language in which the multivariable normal draws, dropout masking, and saveRDS
    assembly are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:d42129f823d16b2cb486d5dc68f9cea59773e813309ee54390e5c5b3b184b045
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuWang-VUMC/stImage
  tools:
  - MUSE
  - R
  - stImage
  - tidyverse
  edam_operation: http://edamontology.org/operation_0244
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# multivariable-normal-expression-generation

## Summary

Generates simulated spatial transcriptomics benchmark data by drawing spot-level gene expression from a multivariable normal distribution (following the MUSE simulation design), embedding distinct or ambiguous spatial patterns, and pairing expression with synthetic image features, spatial coordinates, and ground-truth cluster labels.

## When to use

Use when benchmarking ST integration/clustering methods (e.g. stImage vs MUSE-style baselines) requires synthetic two-modality data with known ground-truth labels, controlled cluster counts, and controlled expression dropout.

## When NOT to use

- The target dataset is real tissue data with no simulation grid needed — use stImage's PreProcessing/MultiModalIntegration workflow on real data instead.
- The goal is to evaluate methods on a single fixed cluster count without dropout stress-testing — the grid of 4/6/8/10 clusters and 60–90% dropout is unnecessary overhead.
- Only image modality or only spatial coordinates are of interest — this skill specifically generates paired expression + image feature two-modality data.

## Inputs

- Simulation grid specification: number of clusters k (4, 6, 8, 10) and dropout rate (60/70/80/90%)
- Spatial pattern choice: distinct or ambiguous spatial patterns
- R environment with stImage and tidyverse installed

## Outputs

- simulated_st_data.rds containing gene expression matrix, image feature profiles, spatial coordinates, and ground-truth cluster labels for every k and dropout-rate combination

## How to apply

Set up an R environment with the stImage package and tidyverse, then define a simulation grid over cluster numbers (k = 4, 6, 8, and 10 clusters, with 10 as the main setting) and dropout rates (60%, 70%, 80%, 90%). Draw spot-level gene expression from a multivariable normal distribution following the MUSE simulation setup, embedding either distinct or ambiguous spatial patterns across spots, and generate matching spot-level image feature profiles so the two modalities mirror the MUSE two-modality design. Apply dropout masking at each specified rate to the expression matrices. Finally assemble the simulated expression matrix, image features, spatial coordinates, and ground-truth cluster labels and save them as simulated_st_data.rds for downstream benchmarking.

## Related tools

- **stImage** (R package environment in which the simulated expression, image features, coordinates, and labels are generated and assembled) — https://github.com/YuWang-VUMC/stImage
- **tidyverse** (Loaded in R for data manipulation while building the simulation grid and assembling the .rds output)
- **MUSE** (Reference simulation setup whose two-modality (expression + image features) design the simulation mirrors)
- **R** (Language in which the multivariable normal draws, dropout masking, and saveRDS assembly are executed)

## Examples

```
Rscript -e "library(tidyverse); # k in c(4,6,8,10); dropout in c(.6,.7,.8,.9); expr <- MASS::mvrnorm(n_spots, mu, Sigma); saveRDS(list(expr, image_features, coords, labels), 'simulated_st_data.rds')"
```

## Evaluation signals

- The saved simulated_st_data.rds loads in R and contains all four components (expression, image features, coordinates, ground-truth labels).
- For every (k, dropout) combination, the number of distinct ground-truth labels equals k.
- Expression matrices show the expected fraction of masked (dropout) entries matching the specified 60/70/80/90% rates.
- Simulated data reproduce both distinct and ambiguous spatial pattern variants, consistent with the MUSE two-modality simulation design.

## Limitations

- The simulation is grounded in the MUSE setup, so realism is limited by that generator's assumptions (multivariable normal expression, synthetic image features) rather than by real histology.
- Only the specific grid values reported in the paper (k = 4, 6, 8, 10 clusters; 60–90% dropout) are validated; other parameter combinations are untested.
- Ground-truth labels are assigned by construction, so they may not reflect biological ambiguity beyond the 'ambiguous spatial pattern' mode explicitly simulated.

## Evidence

- [intro] MUSE simulation design: "MUSE [4], for example, learns joint latent features by integrating morphological image features and transcriptomic data using a self-supervised autoencoder"
- [abstract] stImage as open-source R package environment: "we introduce stImage, an open-source R package that provides a comprehensive and flexible solution for ST analysis"
- [readme] stImage installation grounding for the R environment: [evidence span withheld — anchor status: external_doc]
