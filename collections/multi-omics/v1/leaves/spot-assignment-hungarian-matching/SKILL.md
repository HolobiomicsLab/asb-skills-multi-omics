---
name: spot-assignment-hungarian-matching
description: Use when a simulated SRT dataset (e.g. generated with SRTsim or scDesign3
  via simmethods/simpipe) must be quantitatively compared to a real SRT reference
  in terms of spatial pattern similarity rather than expression-only accuracy.
when_to_use_negative:
- When the goal is general expression-level accuracy evaluation of simulated data
  without spatial coordinates — the paper's accuracy assessment covers multiple metrics
  and the spatial-level metrics are a specific spatial recalibration.
- When the simulated dataset lacks per-spot expression profiles or is not eligible
  for coordinate augmentation, since the Hungarian matching cost is computed from
  expression-profile similarity.
- When comparing non-spatial scRNA-seq simulations, for which no spot coordinates
  or spatial metrics apply.
tools:
- name: simpipe
  role: Standard pipeline providing estimation of parameters from real data, simulation,
    and evaluation (including the spatial-level metric computation with Hungarian
    spot matching)
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Collection of 49 simulation methods used to generate the simulated SRT datasets
    whose spots are matched to the real reference
  repo: https://github.com/duohongrui/simmethods
- name: SRTsim
  role: SRT simulator whose output can serve as the simulated dataset for spatial-level
    accuracy recalibration
- name: scDesign3
  role: SRT simulator whose output can serve as the simulated dataset for spatial-level
    accuracy recalibration
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:a75ecb89e794a5cab2bb5f195b92449fbe9caa0e281ceac9add9944517a97d8c
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Spot assignment via Hungarian matching for spatial-level SRT evaluation

## Summary

Recalculates accuracy scores at the 'spatial-level' for simulated spatially resolved transcriptomics (SRT) data by optimally pairing simulated spots to real spots with the Hungarian algorithm, transferring real X/Y coordinates to simulated spots, and scoring similarity with spatial-multiKS and spatial-KDE metrics.

## When to use

Use when a simulated SRT dataset (e.g. generated with SRTsim or scDesign3 via simmethods/simpipe) must be quantitatively compared to a real SRT reference in terms of spatial pattern similarity rather than expression-only accuracy.

## When NOT to use

- When the goal is general expression-level accuracy evaluation of simulated data without spatial coordinates — the paper's accuracy assessment covers multiple metrics and the spatial-level metrics are a specific spatial recalibration.
- When the simulated dataset lacks per-spot expression profiles or is not eligible for coordinate augmentation, since the Hungarian matching cost is computed from expression-profile similarity.
- When comparing non-spatial scRNA-seq simulations, for which no spot coordinates or spatial metrics apply.

## Inputs

- Real SRT reference dataset with per-spot expression profiles and X/Y spot coordinates
- Simulated SRT dataset generated via simmethods/simpipe (e.g. with SRTsim or scDesign3) with per-spot expression profiles eligible for coordinate augmentation

## Outputs

- matched_spots.csv — complete one-to-one simulated-to-real spot mapping with transferred X/Y coordinate columns
- spatial_metric_scores.csv — numeric spatial-multiKS and spatial-KDE scores

## How to apply

Load the real SRT reference dataset and the simulated SRT dataset (produced through simmethods/simpipe with simulators such as SRTsim or scDesign3), ensuring both carry per-spot expression profiles and that the simulated data are eligible for coordinate augmentation. Solve the optimal one-to-one assignment between simulated and real spots with the Hungarian algorithm, using expression-profile similarity as the matching cost. Transfer the X/Y coordinates of each matched real spot to its paired simulated spot and write the mapping to matched_spots.csv. Then compute the spatial-multiKS score (a multivariate Kolmogorov–Smirnov-type comparison of coordinate-augmented property distributions between real and coordinate-transferred simulated data) and the spatial-KDE score (comparison of kernel density estimates of the spatial patterns after coordinate transfer), collating both values into spatial_metric_scores.csv.

## Related tools

- **simpipe** (Standard pipeline providing estimation of parameters from real data, simulation, and evaluation (including the spatial-level metric computation with Hungarian spot matching)) — https://github.com/duohongrui/simpipe
- **simmethods** (Collection of 49 simulation methods used to generate the simulated SRT datasets whose spots are matched to the real reference) — https://github.com/duohongrui/simmethods
- **SRTsim** (SRT simulator whose output can serve as the simulated dataset for spatial-level accuracy recalibration)
- **scDesign3** (SRT simulator whose output can serve as the simulated dataset for spatial-level accuracy recalibration)

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- matched_spots.csv exists and contains a complete one-to-one simulated-to-real spot mapping including X/Y coordinate columns
- spatial_metric_scores.csv exists and contains numeric entries for both spatial-multiKS and spatial-KDE
- Each simulated spot is paired to exactly one real spot (optimal one-to-one Hungarian assignment, no duplicates)
- Transferred X/Y coordinates in matched_spots.csv come from the paired real spots of the reference dataset
- Both spatial metric values are finite numerics; the article notes that execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations

## Limitations

- Requires the simulated data to be eligible for coordinate augmentation; datasets without compatible per-spot expression profiles cannot be matched
- Execution errors in the broader pipeline are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations
- Users should consider trade-offs between method accuracy and scalability (or functionality) — spatial-level recalculation adds computational cost on top of simulation

## Evidence

- [abstract] SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across various platforms: "SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across various platforms"
- [readme] Simpipe pipeline description: [evidence span withheld — anchor status: not_found]
- [readme] simmethods scope: [evidence span withheld — anchor status: external_doc]
- [methods] simpipe installation: "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [abstract] Execution error causes: "execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations"
