---
name: spatial-coordinate-transfer-to-simulated-spots
description: Use when evaluating spatially resolved transcriptomics simulations (e.g.
  from SRTsim or scDesign3 via simpipe/simmethods) and you need spatial-level accuracy
  scores that require real coordinates to be assigned to simulated spots.
when_to_use_negative:
- The datasets are single-cell (non-spatial) scRNA-seq simulations without spot coordinates
  — general accuracy metrics, not spatial-level metrics, apply
- The simulated dataset cannot accept coordinate augmentation (e.g. no per-spot expression
  profiles to drive Hungarian matching)
- The goal is to compare simulator accuracy performance itself rather than score one
  simulated dataset's spatial similarity to a real reference
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: simpipe
  role: Standard pipeline for estimating parameters from real data, simulating, and
    evaluating simulated data, including the spatial-level evaluation workflow
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Collection of 49 simulation methods used to generate the simulated SRT datasets
    whose spots are coordinate-matched and spatially scored
  repo: https://github.com/duohongrui/simmethods
- name: SRTsim
  role: Best-accuracy simulator for SRT data whose simulated spots can be coordinate-augmented
    for spatial-level evaluation
- name: scDesign3
  role: Best-accuracy simulator for SRT data whose simulated spots can be coordinate-augmented
    for spatial-level evaluation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:5fab2595270f18f87596d7495b7293d2dbfb6972266858cd0552044310fdca56
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# spatial-coordinate-transfer-to-simulated-spots

## Summary

Quantifies the spatial similarity between simulated and real SRT datasets by first solving a one-to-one Hungarian spot matching using expression-profile similarity, transferring matched real X/Y coordinates onto simulated spots, and recomputing accuracy with spatial-level metrics (spatial-KDE and spatial-multiKS).

## When to use

Use when evaluating spatially resolved transcriptomics simulations (e.g. from SRTsim or scDesign3 via simpipe/simmethods) and you need spatial-level accuracy scores that require real coordinates to be assigned to simulated spots.

## When NOT to use

- The datasets are single-cell (non-spatial) scRNA-seq simulations without spot coordinates — general accuracy metrics, not spatial-level metrics, apply
- The simulated dataset cannot accept coordinate augmentation (e.g. no per-spot expression profiles to drive Hungarian matching)
- The goal is to compare simulator accuracy performance itself rather than score one simulated dataset's spatial similarity to a real reference

## Inputs

- Real SRT reference dataset with per-spot expression profiles and X/Y spatial coordinates
- Simulated SRT dataset produced by SRTsim or scDesign3 via simmethods/simpipe, with per-spot expression profiles and eligible for coordinate augmentation

## Outputs

- matched_spots.csv — complete one-to-one simulated-to-real spot mapping with X/Y coordinate columns
- spatial_metric_scores.csv — numeric spatial-multiKS and spatial-KDE entries

## How to apply

Load the real SRT reference dataset and the simulated SRT dataset (generated with SRTsim or scDesign3 under the simmethods/simpipe framework), ensuring both carry per-spot expression profiles and that the simulated data are eligible for coordinate augmentation. Solve the optimal one-to-one assignment between simulated and real spots with the Hungarian algorithm, using expression-profile similarity as the matching cost, then transfer the X/Y coordinates of each matched real spot to its paired simulated spot and write the mapping to matched_spots.csv. Compute the spatial-multiKS score, a multivariate Kolmogorov–Smirnov-type comparison of coordinate-augmented property distributions between real and coordinate-transferred simulated data, and the spatial-KDE score by comparing kernel density estimates of the spatial patterns after coordinate transfer. Collate both values into spatial_metric_scores.csv. The rationale is that SRTsim and scDesign3 are among the best-accuracy simulators across platforms, so their simulated spots need coordinate-aware evaluation to compare spatial patterns against the real reference.

## Related tools

- **simpipe** (Standard pipeline for estimating parameters from real data, simulating, and evaluating simulated data, including the spatial-level evaluation workflow) — https://github.com/duohongrui/simpipe
- **simmethods** (Collection of 49 simulation methods used to generate the simulated SRT datasets whose spots are coordinate-matched and spatially scored) — https://github.com/duohongrui/simmethods
- **SRTsim** (Best-accuracy simulator for SRT data whose simulated spots can be coordinate-augmented for spatial-level evaluation)
- **scDesign3** (Best-accuracy simulator for SRT data whose simulated spots can be coordinate-augmented for spatial-level evaluation)

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- matched_spots.csv exists and contains a complete one-to-one simulated-to-real spot mapping including X/Y coordinate columns
- spatial_metric_scores.csv exists and contains numeric spatial-multiKS and spatial-KDE entries
- The Hungarian assignment is one-to-one: no simulated spot is paired with multiple real spots and vice versa
- Coordinate transfer preserves the real spots' X/Y values exactly for each matched pair (no missing coordinates after transfer)
- Both spatial-KDE and spatial-multiKS scores are finite numbers (non-NaN/non-Inf), avoiding the missing/infinite-value failure mode reported for calculations

## Limitations

- Execution errors in the benchmark are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations, which can break spatial scoring
- The evaluation is only meaningful for simulators whose output is eligible for coordinate augmentation; non-spatial simulations cannot be scored this way
- No method performs best on all criteria, so spatial-level scores should be interpreted together with accuracy, functionality, scalability, and usability trade-offs

## Evidence

- [abstract] Best-accuracy simulators justify spatial evaluation of their output: "SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across various platforms"
- [readme] simpipe's role as the standard evaluation pipeline: [evidence span withheld — anchor status: not_found]
- [methods] Tool installation grounding for the workflow: "devtools::install_github("duohongrui/simmethods") devtools::install_github("duohongrui/simpipe")"
- [abstract] simpipe as the published standard pipeline for simulation and evaluation: "a standard pipeline Simpipe ( https://github.com/duohongrui/simpipe ; https://doi.org/10.5281/zenodo.11178409 )"
