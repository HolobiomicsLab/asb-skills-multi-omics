---
name: spatial-transcriptomics-data-simulation
description: Use when a spatial transcriptomics method (e.g., stImage integrative
  strategies) must be benchmarked against known ground truth, since real ST data lack
  definitive per-spot cluster labels.
when_to_use_negative:
- The analysis targets real spatial transcriptomics datasets (e.g., HER2 breast cancer
  ST or Adult Mouse Olfactory Bulb Visium data) where no reliable ground-truth labels
  exist and empirical evaluation via diagnostic graphs is needed instead
- The task is method development for simulation itself rather than producing labeled
  benchmark data for ARI evaluation of clustering strategies
- Only a single modality (gene expression without matching image features) is needed,
  since the skill's design mirrors MUSE's two-modality setup
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: stImage
  role: R package environment in which the simulated data are generated and later
    used to benchmark integrative strategies
  repo: https://github.com/YuWang-VUMC/stImage
- name: R
  role: Implementation language for the simulation grid, dropout masking, and .rds
    serialization
- name: MUSE
  role: 'Reference simulation design: spot-level expression from a multivariable normal
    distribution plus paired two-modality (expression + image feature) profiles'
- name: tidyverse
  role: R libraries used for data assembly and manipulation during simulation
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:210ea3badbbb84b18160644215e548c877e0ea91b6037811383a07f3e8b8d299
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuWang-VUMC/stImage
  tools:
  - BayesSpace
  - MUSE
  - R
  - SpatialPCA
  - stImage
  - tidyverse
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# spatial-transcriptomics-data-simulation

## Summary

Generate simulated spatial transcriptomics benchmarks in the stImage/MUSE style: spot-level gene expression drawn from a multivariable normal distribution with distinct or ambiguous spatial patterns, matched image feature profiles, spatial coordinates, and ground-truth cluster labels across k = 4, 6, 8, 10 clusters and dropout rates of 60–90%.

## When to use

Use when a spatial transcriptomics method (e.g., stImage integrative strategies) must be benchmarked against known ground truth, since real ST data lack definitive per-spot cluster labels.

## When NOT to use

- The analysis targets real spatial transcriptomics datasets (e.g., HER2 breast cancer ST or Adult Mouse Olfactory Bulb Visium data) where no reliable ground-truth labels exist and empirical evaluation via diagnostic graphs is needed instead
- The task is method development for simulation itself rather than producing labeled benchmark data for ARI evaluation of clustering strategies
- Only a single modality (gene expression without matching image features) is needed, since the skill's design mirrors MUSE's two-modality setup

## Inputs

- Simulation grid specification (cluster numbers k = 4, 6, 8, 10; dropout rates 60%, 70%, 80%, 90%)
- MUSE-style simulation parameters for spot-level multivariable normal gene expression
- Spatial pattern definitions (distinct vs. ambiguous) assigned across spots

## Outputs

- simulated_st_data.rds containing gene expression matrix, image feature matrix, spatial coordinates, and ground-truth cluster labels for each k and dropout-rate combination

## How to apply

In an R environment with the stImage package and tidyverse, define a simulation grid over cluster counts (k = 4, 6, 8, 10) and dropout rates (60%, 70%, 80%, 90%). Generate spot-level gene expression from a multivariable normal distribution following the MUSE simulation setup, embedding both distinct and ambiguous spatial patterns across spots, and generate corresponding spot-level image feature profiles consistent with the assigned patterns to mirror MUSE's two-modality design. Apply dropout masking at each specified rate to the expression matrices, then assemble gene expression, image features, spatial coordinates, and ground-truth cluster labels into a single object saved as simulated_st_data.rds. Validate by re-loading the .rds in R and confirming all four components exist for every k and dropout-rate combination and that the number of distinct labels equals k; the labels then serve as ground truth for ARI benchmarking of integrative strategies such as WNN, Spectrum, MCIA, and tICA.

## Related tools

- **stImage** (R package environment in which the simulated data are generated and later used to benchmark integrative strategies) — https://github.com/YuWang-VUMC/stImage
- **R** (Implementation language for the simulation grid, dropout masking, and .rds serialization)
- **MUSE** (Reference simulation design: spot-level expression from a multivariable normal distribution plus paired two-modality (expression + image feature) profiles)
- **tidyverse** (R libraries used for data assembly and manipulation during simulation)

## Examples

```
library(stImage); library(tidyverse); sim <- readRDS("simulated_st_data.rds"); stopifnot(length(unique(sim$labels)) == k)
```

## Evaluation signals

- The saved simulated_st_data.rds loads in R without error
- The object contains all four components (gene expression, image features, spatial coordinates, ground-truth labels) for every k and dropout-rate combination
- For each combination, the number of distinct ground-truth labels equals the specified k (4, 6, 8, or 10)
- Downstream ARI benchmarking shows the expected ordering: modality integration outperforms single-modality strategies, with WNN and Spectrum achieving higher ARI than MCIA and tICA, especially under high dropout rates

## Limitations

- No single integration approach performed well on every dataset in the article, so simulated-data rankings may not transfer to real tissue data
- The simulation follows the MUSE two-modality setup and its assumptions; patterns not representable under the multivariable-normal/dropout model are not covered
- Multi-modal integration is computationally more intensive and algorithm-dependent, which constrains how large a simulation grid can be swept in practice

## Evidence

- [intro] MUSE two-modality simulation design: "MUSE [4], for example, learns joint latent features by integrating morphological image features and transcriptomic data using a self-supervised autoencoder"
- [abstract] stImage open-source R package for ST analysis: "we introduce stImage, an open-source R package that provides a comprehensive and flexible solution for ST analysis"
- [readme] stImage availability and environment: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images."
