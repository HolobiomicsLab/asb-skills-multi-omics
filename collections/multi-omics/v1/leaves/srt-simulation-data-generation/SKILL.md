---
name: srt-simulation-data-generation
description: Use when you need to simulate SRT data from a real spatial reference
  dataset and then recalculate accuracy scores that specifically quantify how similar
  the simulated spots are to the real ones in spatial terms.
when_to_use_negative:
- The input data are pure scRNA-seq without spatial coordinates — the spatial-level
  Hungarian matching and coordinate-transfer metrics require real spots with X/Y positions
- The goal is non-spatial simulation scenarios (groups, batches, trajectories) where
  general-level accuracy metrics suffice and no real SRT reference is available for
  spot matching
- The user only needs to estimate parameters and simulate without benchmarking similarity
  — in that case direct simmethods calls (or the simshiny interface) without the spatial
  evaluation step are appropriate
tools:
- name: SRTsim
  role: Top-accuracy simulator used to generate the simulated SRT dataset from the
    real reference
  repo: https://github.com/duohongrui/simpipe
- name: scDesign3
  role: Alternative top-accuracy simulator used to generate the simulated SRT dataset
    for comparison
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: R package bundling 49 simulation methods for single-cell and spatial transcriptome
    data used to generate the simulated data
  repo: https://github.com/duohongrui/simmethods
- name: simpipe
  role: Standard pipeline for estimating parameters from real data, simulating datasets,
    and evaluating them; produces matched_spots.csv and spatial_metric_scores.csv
  repo: https://github.com/duohongrui/simpipe
- name: simpipe2docker
  role: Links the R environment to the duohongrui/simpipe Docker container for running
    the simulation and evaluation workflow reproducibly
  repo: https://github.com/duohongrui/simpipe2docker
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:a3375fffc11ca66467fd0a052090560e83d7eecb6d8c8d33c076f727824c7a87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simpipe
  tools:
  - SRTsim
  - scDesign3
  - simmethods
  - simpipe
  - simpipe2docker
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3
n_claims: 10
---

# srt-simulation-data-generation

## Summary

Generates simulated spatially resolved transcriptomics (SRT) datasets from real reference data using the simpipe/simmethods framework (e.g. SRTsim, scDesign3), and quantifies spatial-level similarity to the real data via Hungarian spot matching with spatial-multiKS and spatial-KDE scores.

## When to use

Use when you need to simulate SRT data from a real spatial reference dataset and then recalculate accuracy scores that specifically quantify how similar the simulated spots are to the real ones in spatial terms.

## When NOT to use

- The input data are pure scRNA-seq without spatial coordinates — the spatial-level Hungarian matching and coordinate-transfer metrics require real spots with X/Y positions
- The goal is non-spatial simulation scenarios (groups, batches, trajectories) where general-level accuracy metrics suffice and no real SRT reference is available for spot matching
- The user only needs to estimate parameters and simulate without benchmarking similarity — in that case direct simmethods calls (or the simshiny interface) without the spatial evaluation step are appropriate

## Inputs

- Real SRT reference dataset with per-spot expression profiles and X/Y spatial coordinates
- Simulated SRT dataset generated with SRTsim or scDesign3 through the simpipe/simmethods framework, eligible for coordinate augmentation

## Outputs

- matched_spots.csv — complete one-to-one simulated-to-real spot mapping with transferred X/Y coordinate columns
- spatial_metric_scores.csv — numeric spatial-multiKS and spatial-KDE scores
- Simulated SRT count matrix with per-spot expression profiles

## How to apply

Install simmethods and simpipe from GitHub with devtools (or pull the duohongrui/simpipe Docker image and drive it via simpipe2docker), then load the real SRT reference dataset and the simulated dataset produced by a chosen simulator such as SRTsim or scDesign3, ensuring both carry per-spot expression profiles and the simulated data are eligible for coordinate augmentation. Solve the optimal one-to-one assignment between simulated and real spots with the Hungarian algorithm, using expression-profile similarity as the matching cost, and transfer the X/Y coordinates of each matched real spot to its paired simulated spot (matched_spots.csv). Compute the spatial-multiKS score — a multivariate Kolmogorov–Smirnov-type comparison of coordinate-augmented property distributions between real and coordinate-transferred simulated data — and the spatial-KDE score by comparing kernel density estimates of the spatial patterns after coordinate transfer, collating both into spatial_metric_scores.csv. Rationale: the article's benchmark found SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across various platforms, so these are preferred generators; because some scRNA-seq methods have potential compatibility for simulating SRT data, spatial metrics that require coordinate transfer are what distinguish spatial fidelity from expression-level fidelity.

## Related tools

- **SRTsim** (Top-accuracy simulator used to generate the simulated SRT dataset from the real reference) — https://github.com/duohongrui/simpipe
- **scDesign3** (Alternative top-accuracy simulator used to generate the simulated SRT dataset for comparison) — https://github.com/duohongrui/simpipe
- **simmethods** (R package bundling 49 simulation methods for single-cell and spatial transcriptome data used to generate the simulated data) — https://github.com/duohongrui/simmethods
- **simpipe** (Standard pipeline for estimating parameters from real data, simulating datasets, and evaluating them; produces matched_spots.csv and spatial_metric_scores.csv) — https://github.com/duohongrui/simpipe
- **simpipe2docker** (Links the R environment to the duohongrui/simpipe Docker container for running the simulation and evaluation workflow reproducibly) — https://github.com/duohongrui/simpipe2docker

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- matched_spots.csv exists and contains a complete one-to-one simulated-to-real spot mapping with X/Y coordinate columns
- spatial_metric_scores.csv contains numeric spatial-multiKS and spatial-KDE entries
- The Hungarian assignment yields a bijection (each simulated spot paired with exactly one real spot), so the coordinate transfer is invertible and lossless
- Simulators that score high on spatial-level metrics should align with the benchmark finding that SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across platforms
- Coordinate-augmented simulated spots, after transfer, should reproduce spatial property distributions of the real data closely enough that the multiKS/KDE comparison is meaningful (non-NaN scores)

## Limitations

- No method performs best on all criteria, so a good-yet-not-the-best simulator may need to be chosen if it solves the problem effectively and reasonably; users should weigh accuracy against scalability and functionality
- Execution errors during simulation are mainly caused by failed parameter estimations and the appearance of missing or infinite values in calculations
- Some scRNA-seq simulation methods only have potential compatibility for SRT data, and excessively flexible parameter settings can distort simulated data so it does not reflect real biological differences

## Evidence

- [abstract] SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across various platforms: "SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across various platforms"
- [abstract] standard pipeline Simpipe repo: "a standard pipeline Simpipe ( https://github.com/duohongrui/simpipe ; https://doi.org/10.5281/zenodo.11178409 )"
- [abstract] scRNA-seq methods compatible with SRT: "some methods tailored to scRNA-seq data have potential compatibility for simulating SRT data"
- [readme] simmethods collection description: [evidence span withheld — anchor status: external_doc]
- [methods] install commands: "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
