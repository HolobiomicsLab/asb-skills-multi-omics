---
name: image-feature-profile-simulation
description: Use when benchmarking multi-modal ST integration strategies requires
  synthetic paired expression + image-feature data with controlled cluster counts
  and dropout rates and trusted ground-truth labels.
when_to_use_negative:
- The analysis uses real tissue data (e.g., HER2 breast cancer ST or Adult Mouse Olfactory
  Bulb Visium) where empirical histology images — not simulated image feature profiles
  — are available
- The benchmark question concerns single-modality (gene-expression-only) clustering,
  where simulated paired image features add unnecessary complexity
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3383
tools:
- name: stImage
  role: R package environment in which the simulated expression, image features, coordinates,
    and labels are generated and later used to benchmark integrative strategies
  repo: https://github.com/YuWang-VUMC/stImage
- name: MUSE
  role: 'Reference design being mirrored: spot-level gene expression is simulated
    from a multivariable normal distribution and paired with image features following
    the MUSE two-modality setup'
- name: tidyverse
  role: R toolkit used inside the simulation workflow (loaded via library(tidyverse))
    for data assembly and manipulation
- name: R
  role: Language used to run the simulation, apply dropout masking, and save simulated_st_data.rds
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:f7a555341580f2ad5cc6b0c89b3e832670aa67bcd94d58471e1fd4de45fe857d
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
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Image-feature-profile simulation

## Summary

A simulation skill that generates paired spot-level gene expression and spot-level image feature profiles with known ground-truth spatial patterns, modeled on the MUSE two-modality design, for benchmarking ST integration methods such as stImage.

## When to use

Use when benchmarking multi-modal ST integration strategies requires synthetic paired expression + image-feature data with controlled cluster counts and dropout rates and trusted ground-truth labels.

## When NOT to use

- The analysis uses real tissue data (e.g., HER2 breast cancer ST or Adult Mouse Olfactory Bulb Visium) where empirical histology images — not simulated image feature profiles — are available
- The benchmark question concerns single-modality (gene-expression-only) clustering, where simulated paired image features add unnecessary complexity

## Inputs

- Simulation grid specification (cluster counts k = 4, 6, 8, 10; dropout rates 60/70/80/90%)
- Spatial pattern definitions (distinct vs. ambiguous) per spot
- stImage R package environment with tidyverse

## Outputs

- simulated_st_data.rds containing spot-level gene expression matrix, spot-level image feature profiles, spatial coordinates, and ground-truth cluster labels for each k and dropout-rate combination

## How to apply

Load the stImage R environment with tidyverse and define a simulation grid over cluster numbers (k = 4, 6, 8, 10; 10 is the main setting) and expression dropout rates (60%, 70%, 80%, 90%). Generate spot-level gene expression from a multivariable normal distribution following the MUSE simulation setup, embedding both distinct and ambiguous spatial patterns across spots, then generate a corresponding spot-level image feature profile consistent with each assigned spatial pattern so the two modalities mirror the MUSE two-modality design. Apply dropout masking at each specified rate to the simulated expression matrices. Assemble expression, image features, spatial coordinates, and ground-truth cluster labels into a single simulated_st_data.rds object. Validate by reloading the RDS and confirming all four components exist for every k/dropout combination and that the number of distinct labels equals k.

## Related tools

- **stImage** (R package environment in which the simulated expression, image features, coordinates, and labels are generated and later used to benchmark integrative strategies) — https://github.com/YuWang-VUMC/stImage
- **MUSE** (Reference design being mirrored: spot-level gene expression is simulated from a multivariable normal distribution and paired with image features following the MUSE two-modality setup)
- **tidyverse** (R toolkit used inside the simulation workflow (loaded via library(tidyverse)) for data assembly and manipulation)
- **R** (Language used to run the simulation, apply dropout masking, and save simulated_st_data.rds)

## Examples

```
library(tidyverse); sim <- generate_simulated_st(k = c(4,6,8,10), dropout = c(0.6,0.7,0.8,0.9)); saveRDS(sim, 'simulated_st_data.rds')
```

## Evaluation signals

- simulated_st_data.rds reloads in R without error and exposes all four components: expression, image features, spatial coordinates, ground-truth labels
- For every simulated combination, the number of distinct ground-truth labels equals the configured k (4, 6, 8, or 10)
- Every k/dropout-rate combination (dropout in {60%, 70%, 80%, 90%}) is present and complete
- Simulated image feature profiles are consistent with the assigned spatial pattern class (distinct vs. ambiguous) per spot
- Dropout masking yields expression matrices whose missingness matches the specified rate

## Limitations

- No single integration approach performed well on every dataset tested, so simulation outcomes must be interpreted per dataset rather than generalized
- Multi-modal integration is computationally more intensive and algorithm-dependent, which affects how many simulation combinations can be feasibly benchmarked
- Simulated spatial patterns are limited to the distinct vs. ambiguous dichotomy of the MUSE-style setup and may not capture real tissue histology variability

## Evidence

- [intro] MUSE design basis: "MUSE [4], for example, learns joint latent features by integrating morphological image features and transcriptomic data using a self-supervised autoencoder"
- [abstract] stImage modality scope: "By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information."
- [readme] R environment: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images."
- [discussion] Computational cost caveat: "While multi-modal integration is computationally more intensive and algorithm-dependent, the overall runtime and resource usage of stImage remain manageable even in a typical office desktop"
