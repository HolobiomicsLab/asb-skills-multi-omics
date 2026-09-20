---
name: spatial-kde-density-evaluation
description: Use when you have a simulated SRT dataset (e.g. generated with SRTsim
  or scDesign3 through simmethods/simpipe) plus a real SRT reference with per-spot
  expression and X/Y coordinates, and you need spatial-level accuracy scores beyond
  non-spatial metrics.
when_to_use_negative:
- The datasets lack spatial X/Y coordinates (e.g. non-spatial scRNA-seq simulations)
  — coordinate augmentation and KDE comparison cannot be performed
- Only expression-level (non-spatial) accuracy metrics are needed, so the Hungarian
  matching/coordinate-transfer overhead is unnecessary
- The simulated spots cannot be paired one-to-one with real spots, breaking the coordinate-transfer
  step
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: simpipe
  role: Standard pipeline hosting the spatial-level evaluation (Hungarian spot matching,
    coordinate transfer, spatial-KDE and spatial-multiKS scoring)
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Collection of 49 simulation methods used to generate the simulated SRT datasets
    that are evaluated by the spatial metrics
  repo: https://github.com/duohongrui/simmethods
- name: SRTsim
  role: One of the top-accuracy SRT simulation methods whose output is fed into the
    spatial-level evaluation
- name: scDesign3
  role: One of the top-accuracy SRT simulation methods whose output is fed into the
    spatial-level evaluation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:874c485ad3ae5a7ca8154d76b6b9efe85b597920ae7a58b3ee15f92516188552
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simpipe
  tools:
  - SRTsim
  - scDesign3
  - simmethods
  - simpipe
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# spatial-kde-density-evaluation

## Summary

A spatial-level accuracy evaluation skill from the simpipe benchmark that quantifies similarity between simulated and real spatially resolved transcriptomics (SRT) datasets. It transfers real X/Y coordinates onto simulated spots via Hungarian spot matching and recalculates accuracy as spatial-KDE and spatial-multiKS scores.

## When to use

Use when you have a simulated SRT dataset (e.g. generated with SRTsim or scDesign3 through simmethods/simpipe) plus a real SRT reference with per-spot expression and X/Y coordinates, and you need spatial-level accuracy scores beyond non-spatial metrics.

## When NOT to use

- The datasets lack spatial X/Y coordinates (e.g. non-spatial scRNA-seq simulations) — coordinate augmentation and KDE comparison cannot be performed
- Only expression-level (non-spatial) accuracy metrics are needed, so the Hungarian matching/coordinate-transfer overhead is unnecessary
- The simulated spots cannot be paired one-to-one with real spots, breaking the coordinate-transfer step

## Inputs

- Real SRT reference dataset with per-spot expression profiles and X/Y spatial coordinates
- Simulated SRT dataset generated with SRTsim or scDesign3 via the simmethods/simpipe framework (eligible for coordinate augmentation)

## Outputs

- matched_spots.csv — complete one-to-one simulated-to-real spot mapping with X/Y coordinate columns
- spatial_metric_scores.csv — numeric spatial-multiKS and spatial-KDE entries

## How to apply

Load both the real SRT reference and the simulated dataset so that each spot carries a per-spot expression profile, and confirm the simulated data are eligible for coordinate augmentation. Solve the optimal one-to-one assignment between simulated and real spots with the Hungarian algorithm, using expression-profile similarity as the matching cost, then transfer the X/Y coordinates of each matched real spot to its paired simulated spot and write the mapping to matched_spots.csv. Compute the spatial-multiKS score — a multivariate Kolmogorov–Smirnov-type comparison of coordinate-augmented property distributions between real and coordinate-transferred simulated data — and the spatial-KDE score by comparing kernel density estimates of the spatial patterns after coordinate transfer. Collate both numeric values into spatial_metric_scores.csv. This follows the simpipe pipeline, whose stated goal is estimating parameters from real datasets, simulating, and evaluating simulated data from general and functional perspectives; in the underlying benchmark, SRTsim and scDesign3 are among the top accuracy performers across platforms.

## Related tools

- **simpipe** (Standard pipeline hosting the spatial-level evaluation (Hungarian spot matching, coordinate transfer, spatial-KDE and spatial-multiKS scoring)) — https://github.com/duohongrui/simpipe
- **simmethods** (Collection of 49 simulation methods used to generate the simulated SRT datasets that are evaluated by the spatial metrics) — https://github.com/duohongrui/simmethods
- **SRTsim** (One of the top-accuracy SRT simulation methods whose output is fed into the spatial-level evaluation)
- **scDesign3** (One of the top-accuracy SRT simulation methods whose output is fed into the spatial-level evaluation)

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- matched_spots.csv exists and contains a complete one-to-one simulated-to-real spot mapping including X/Y coordinate columns
- spatial_metric_scores.csv exists and contains numeric entries for both spatial-multiKS and spatial-KDE
- Both real and simulated inputs carry per-spot expression profiles and the simulated data passed the coordinate-augmentation eligibility check before matching
- The Hungarian assignment yields a bijective (one-to-one) spot pairing rather than many-to-many matches
- Scores are comparable across simulated datasets (e.g. SRTsim vs scDesign3 outputs) since they are recalculated on the same real reference

## Limitations

- Scores depend on the quality of the expression-similarity-based Hungarian matching; failed parameter estimation or missing/infinite values in calculations — the main causes of execution errors in the benchmark — can break the pipeline
- No single method performs best on all criteria, so spatial-KDE/spatial-multiKS results should be weighed together with scalability and functionality trade-offs when choosing a simulator
- Coordinate transfer assumes the simulated spots can meaningfully inherit real spatial positions; it is a similarity quantification, not proof of spatial causality

## Evidence

- [readme] simpipe evaluates simulated data: "The goal of simpipe is to establish a standard pipeline to estimating parameters from real datasets, simulating single-cell RNA-seq datasets and evaluating the simulated data from general and"
- [abstract] top accuracy performers: "SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across various platforms"
- [methods] installation: "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [abstract] benchmark scope: "We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability"
