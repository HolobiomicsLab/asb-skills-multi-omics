---
name: simulation-fidelity-metric-computation
description: Use when you have a real reference scRNA-seq/SRT expression matrix plus
  simulated counterpart matrices and need quantitative accuracy scores of simulation
  fidelity across data properties.
when_to_use_negative:
- You only need scalability, usability, or functionality scores of simulation methods
  — this skill covers the accuracy/fidelity axis only
- You do not yet have simulated matrices; parameter estimation and simulation must
  be done first (e.g., via simmethods/simpipe simulation steps)
- Input is SRT data with trajectory information and tislingshot has not been installed
  for the simulation step
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0203
tools:
- name: simpipe
  role: Standard pipeline that computes the 15 data properties and the eight distance
    metrics comparing real vs simulated expression matrices
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Provides the 49 simulation methods that generate the simulated expression
    matrices and vignette example matrices evaluated against the real reference
  repo: https://github.com/duohongrui/simmethods
- name: devtools
  role: Used to install simmethods and simpipe from GitHub
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:c6891a568c8e5452b151f231347f5900d07b1762147ff7dacdf854d90bdf4030
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simpipe
  tools:
  - devtools
  - simmethods
  - simpipe
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# simulation-fidelity-metric-computation

## Summary

Computes accuracy metrics quantifying how well simulated scRNA-seq/SRT data reproduce real data, by evaluating 15 data properties (e.g., CCC, FCO, LS) on real vs simulated expression matrices with six 1-D and two 2-D distance metrics via the simpipe R package.

## When to use

Use when you have a real reference scRNA-seq/SRT expression matrix plus simulated counterpart matrices and need quantitative accuracy scores of simulation fidelity across data properties.

## When NOT to use

- You only need scalability, usability, or functionality scores of simulation methods — this skill covers the accuracy/fidelity axis only
- You do not yet have simulated matrices; parameter estimation and simulation must be done first (e.g., via simmethods/simpipe simulation steps)
- Input is SRT data with trajectory information and tislingshot has not been installed for the simulation step

## Inputs

- Real reference scRNA-seq or SRT expression matrix (e.g., Simpipe vignette example matrix)
- Simulated expression matrices produced by Simmethods (one or more methods/parameter settings)

## Outputs

- metric_scores.csv with one row per data-property x metric (15 properties x 6 1-D metrics + 2 2-D metrics) per simulated dataset, all values finite

## How to apply

Install simmethods and simpipe from GitHub with devtools::install_github(). Load the Simpipe vignette real reference matrix and the Simmethods-simulated counterpart matrices. For each of the 15 data properties (LS, FZC, CCC, TMM, ELS, FCO, RLZ, ME, SD, CV, FZG, FGO, RMS, RMZ, RDM), compute the property vector on the real and every simulated matrix. For 1-D (per-cell/per-gene) properties compute MAD, KS, MAE, RMSE, OV, and BH distances between real and simulated distributions; for 2-D (pairwise/correlation) properties compute the multiKS statistic and KDE-based distance. Assemble one row per data-property x metric per simulated dataset and check that all values are finite, since missing/infinite values are the paper's main source of execution errors.

## Related tools

- **simpipe** (Standard pipeline that computes the 15 data properties and the eight distance metrics comparing real vs simulated expression matrices) — https://github.com/duohongrui/simpipe
- **simmethods** (Provides the 49 simulation methods that generate the simulated expression matrices and vignette example matrices evaluated against the real reference) — https://github.com/duohongrui/simmethods
- **devtools** (Used to install simmethods and simpipe from GitHub)

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- metric_scores.csv exists and contains rows for all 15 data properties (LS, FZC, CCC, TMM, ELS, FCO, RLZ, ME, SD, CV, FZG, FGO, RMS, RMZ, RDM) crossed with the six 1-D metrics (MAD, KS, MAE, RMSE, OV, BH) and the two 2-D metrics (multiKS, KDE-based distance)
- Every metric value in the output table is finite — no missing or infinite values, which the paper identifies as the main source of execution errors
- Each simulated dataset has a complete row set (15 properties x 8 metrics), enabling per-method comparison of accuracy
- Property vectors were computed identically on the real reference matrix and every simulated matrix, so distances are comparable

## Limitations

- Execution errors are mainly caused by failed parameter estimations and the appearance of missing or infinite values in calculations
- No method performs best on all criteria, so metric scores must be interpreted as trade-offs rather than a single ranking
- Confounding technical and biological effects can prevent methods from accurately capturing heterogeneity, limiting fidelity scores for such datasets

## Evidence

- [methods] Accuracy metrics and 15 data properties: "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [readme] Pipeline scope: "The goal of simpipe is to establish a standard pipeline to estimating parameters from real datasets, simulating single-cell RNA-seq datasets and evaluating the simulated data"
- [abstract] Error source: "execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations"
- [abstract] Benchmark scope: "We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability using 152 reference datasets derived from 24"
