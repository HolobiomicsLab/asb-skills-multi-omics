---
name: synthetic-benchmark-ground-truth-labeling
description: 'Use when an integrative ST method (e.g., stImage''s 54 strategies) must
  be benchmarked quantitatively: synthetic expression plus image features are needed
  with known cluster counts (k = 4, 6, 8, 10) and known distinct vs. ambiguous spatial
  patterns.'
when_to_use_negative:
- The dataset already carries expert-annotated regions or ground-truth labels, making
  simulation unnecessary
- The goal is a real-tissue case study (e.g., HER2 breast cancer ST or Adult Mouse
  Olfactory Bulb Visium data) rather than controlled benchmarking
- Only one modality (expression alone) is being analyzed and no two-modality benchmark
  comparison is intended
tools:
- name: stImage
  role: R environment in which the simulated expression, image features, coordinates,
    and labels are generated and later benchmarked via its integrative strategies
  repo: https://github.com/YuWang-VUMC/stImage
- name: tidyverse
  role: Loaded in R to support data assembly and manipulation of the simulated matrices
- name: MUSE
  role: 'Provides the reference simulation design: multivariable-normal expression
    plus paired image features as two modalities'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:6fd2976d72a28fefb6223c1ab77590a6866dcf0494d4f6e28a330ebba70deedf
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
yes_rate: 0.5
n_claims: 14
---

# synthetic-benchmark-ground-truth-labeling

## Summary

Generates simulated spatial transcriptomics benchmarks with matched gene expression, image features, spatial coordinates, and ground-truth cluster labels (MUSE-style two-modality design), enabling quantitative evaluation of integrative methods like stImage against known spatial structure.

## When to use

Use when an integrative ST method (e.g., stImage's 54 strategies) must be benchmarked quantitatively: synthetic expression plus image features are needed with known cluster counts (k = 4, 6, 8, 10) and known distinct vs. ambiguous spatial patterns.

## When NOT to use

- The dataset already carries expert-annotated regions or ground-truth labels, making simulation unnecessary
- The goal is a real-tissue case study (e.g., HER2 breast cancer ST or Adult Mouse Olfactory Bulb Visium data) rather than controlled benchmarking
- Only one modality (expression alone) is being analyzed and no two-modality benchmark comparison is intended

## Inputs

- Simulation grid parameters (k = 4/6/8/10 clusters; dropout rates 60/70/80/90%)
- MUSE simulation setup (multivariable normal distribution for spot-level expression)
- stImage R package environment with tidyverse

## Outputs

- simulated_st_data.rds containing gene expression matrix, image feature profiles, spatial coordinates, and ground-truth cluster labels per k and dropout-rate combination

## How to apply

Define a simulation grid over cluster counts (k = 4, 6, 8, 10; k = 10 is the main setting in stImage) and dropout rates (60%, 70%, 80%, 90%). Generate spot-level gene expression from a multivariable normal distribution following the MUSE simulation setup, embedding both distinct and ambiguous spatial patterns across spots, then generate matching spot-level image feature profiles consistent with the assigned patterns to mirror MUSE's two-modality design. Apply dropout masking at each rate to the simulated expression matrices. Assemble expression, image features, spatial coordinates, and ground-truth labels into a single RDS artifact (simulated_st_data.rds) so downstream tools like stImage can be scored against the true labels. The rationale is that real ST data lack ground truth, so controlled synthetic patterns isolate how well each integrative strategy recovers distinct vs. ambiguous tissue structures.

## Related tools

- **stImage** (R environment in which the simulated expression, image features, coordinates, and labels are generated and later benchmarked via its integrative strategies) — https://github.com/YuWang-VUMC/stImage
- **tidyverse** (Loaded in R to support data assembly and manipulation of the simulated matrices)
- **MUSE** (Provides the reference simulation design: multivariable-normal expression plus paired image features as two modalities)

## Examples

```
saveRDS(list(expression = expr, image_features = img, coordinates = coords, labels = labels), "simulated_st_data.rds")
```

## Evaluation signals

- simulated_st_data.rds loads in R without error and exposes all four components (expression, image features, coordinates, ground-truth labels)
- For every k and dropout-rate combination, the number of distinct ground-truth labels equals k
- Both distinct and ambiguous spatial pattern variants are present in the generated set
- Expression matrices reflect the specified dropout rates (60/70/80/90%) after masking
- Image feature profiles are consistent with the assigned spatial patterns at each spot, mirroring the MUSE two-modality design

## Limitations

- Simulated data follow the MUSE two-modality design and may not capture all real-tissue complexities (e.g., histology quality variation or conflicting image-expression signals highlighted in stImage's discussion)
- Only four cluster counts (4, 6, 8, 10) and four dropout rates (60-90%) are covered; other regimes require extending the grid
- Ground-truth labels are valid only for these synthetic datasets and cannot be transferred to real ST benchmarks where no single integration approach performed well in every dataset

## Evidence

- [methods] Simulated ST datasets were generated exhibiting either distinct or ambiguous spatial patterns, with varying numbers of clusters (10 clusters in the main setting; 4, 6, and 8 clusters in additional settings).: [evidence span withheld — anchor status: not_found]
- [methods] Generate spot-level gene expression from a multivariable normal distribution following the MUSE simulation setup, embedding both distinct and ambiguous spatial patterns across spots.: [evidence span withheld — anchor status: not_found]
- [methods] Assemble the simulated gene expression, image features, spatial coordinates, and ground-truth cluster labels and save them as simulated_st_data.rds.: [evidence span withheld — anchor status: not_found]
- [readme] StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images.: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images."
