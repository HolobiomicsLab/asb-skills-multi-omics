---
name: spatial-multiks-distribution-comparison
description: Use when benchmarking simulated spatially resolved transcriptomics (SRT)
  datasets against a real SRT reference and you need a spatial-level accuracy score
  that quantifies similarity of spatial property distributions after spot matching.
when_to_use_negative:
- The dataset lacks spatial coordinates or the simulated data cannot be coordinate-augmented
  via Hungarian spot matching, so no spatial-level comparison is possible
- You need general (non-spatial) simulation accuracy metrics for scRNA-seq data rather
  than SRT spatial fidelity scores
- The comparison target is method scalability or usability rather than accuracy of
  simulated versus real SRT data
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_3308
tools:
- name: simpipe
  role: standard pipeline that estimates parameters from real data, simulates datasets,
    and evaluates simulated data from general and functional perspectives, including
    the spatial-level metric workflow
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: collection of 49 simulation methods used to generate the simulated SRT datasets
    whose similarity to real SRT data is scored
  repo: https://github.com/duohongrui/simmethods
- name: SRTsim
  role: SRT simulation method among the best-accuracy performers; its output serves
    as input to the spatial-multiKS comparison
- name: scDesign3
  role: SRT simulation method among the best-accuracy performers; its output serves
    as input to the spatial-multiKS comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:75e3b459eadac8658cb05b06cc5e1358e9feed0c7a573fa6780a8d811b31458a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simpipe
  tools:
  - SRTsim
  - scDesign3
  - simmethods
  - simpipe
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# spatial-multiks-distribution-comparison

## Summary

Computes the spatial-multiKS score, a multivariate Kolmogorov–Smirnov-type comparison of coordinate-augmented property distributions between real SRT data and simulated SRT data, as one of the benchmark's 'spatial-level' accuracy metrics.

## When to use

Use when benchmarking simulated spatially resolved transcriptomics (SRT) datasets against a real SRT reference and you need a spatial-level accuracy score that quantifies similarity of spatial property distributions after spot matching.

## When NOT to use

- The dataset lacks spatial coordinates or the simulated data cannot be coordinate-augmented via Hungarian spot matching, so no spatial-level comparison is possible
- You need general (non-spatial) simulation accuracy metrics for scRNA-seq data rather than SRT spatial fidelity scores
- The comparison target is method scalability or usability rather than accuracy of simulated versus real SRT data

## Inputs

- Real SRT reference dataset with per-spot expression profiles and X/Y coordinates
- Simulated SRT dataset generated with SRTsim or scDesign3 through the simmethods/simpipe framework, eligible for coordinate augmentation
- matched_spots.csv containing a one-to-one simulated-to-real spot mapping with X/Y coordinate columns

## Outputs

- Numeric spatial-multiKS score in spatial_metric_scores.csv
- Paired spatial-KDE score in spatial_metric_scores.csv

## How to apply

Load the real SRT reference and the simulated SRT data generated via simmethods/simpipe (e.g., with SRTsim or scDesign3), both carrying per-spot expression profiles. Solve the optimal one-to-one assignment between simulated and real spots using the Hungarian algorithm with expression-profile similarity as the matching cost, then transfer the X/Y coordinates of each matched real spot to its paired simulated spot and write the mapping to matched_spots.csv. With coordinates augmented, compute the spatial-multiKS score as a multivariate Kolmogorov–Smirnov-type comparison of the property distributions between real and coordinate-transferred simulated data. In the benchmark, spatial-multiKS is computed alongside spatial-KDE and the pair is collated into spatial_metric_scores.csv for recalculation of accuracy scores. Correct application hinges on the coordinate transfer being complete and one-to-one before the KS-type comparison is run.

## Related tools

- **simpipe** (standard pipeline that estimates parameters from real data, simulates datasets, and evaluates simulated data from general and functional perspectives, including the spatial-level metric workflow) — https://github.com/duohongrui/simpipe
- **simmethods** (collection of 49 simulation methods used to generate the simulated SRT datasets whose similarity to real SRT data is scored) — https://github.com/duohongrui/simmethods
- **SRTsim** (SRT simulation method among the best-accuracy performers; its output serves as input to the spatial-multiKS comparison)
- **scDesign3** (SRT simulation method among the best-accuracy performers; its output serves as input to the spatial-multiKS comparison)

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- spatial_metric_scores.csv exists and contains numeric spatial-multiKS and spatial-KDE entries
- matched_spots.csv contains a complete one-to-one simulated-to-real spot mapping with X/Y coordinate columns before the metric is computed
- Both matched_spots.csv and spatial_metric_scores.csv exist as validation outputs of the workflow
- The simulated data were generated through the simmethods/simpipe framework and are eligible for coordinate augmentation
- The Hungarian matching used expression-profile similarity as the one-to-one assignment cost, not arbitrary spot indices

## Limitations

- The metric depends on Hungarian spot matching quality; a poor expression-based one-to-one assignment distorts the coordinate transfer and hence the multiKS score
- Per the benchmark's own guidance, no method performs best on all criteria, so spatial-multiKS should be interpreted together with other accuracy, functionality, scalability, and usability metrics
- Execution errors in the surrounding pipeline are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations

## Evidence

- [abstract] SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across various platforms: "SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across various platforms"
- [methods] devtools::install_github("duohongrui/simmethods")\ndevtools::install_github("duohongrui/simpipe"): "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [readme] The goal of simpipe is to establish a standard pipeline to estimating parameters from real datasets, simulating single-cell RNA-seq datasets and evaluating the simulated data from general and functional perspectives.: "The goal of simpipe is to establish a standard pipeline to estimating parameters from real datasets, simulating single-cell RNA-seq datasets and evaluating the simulated data from general and"
- [readme] Simmethods collects and documents 49 popular and common simulation methods for single-cell and spatial transcriptome data.: [evidence span withheld — anchor status: external_doc]
