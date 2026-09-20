---
name: zero-inflation-and-library-size-characterization
description: Use when judging whether simulated single-cell or spatial transcriptomics
  matrices reproduce real-data sparsity (zero proportions) and library-size distributions,
  e.g., scoring FZC, FZG, FGO, LS, RLZ against a real reference.
when_to_use_negative:
- When the goal is functional evaluation (e.g., clustering or trajectory ground-truth
  fidelity) rather than accuracy on data properties such as sparsity and library size
- When only summary statistics (mean expression, variance) of the simulation are needed
  rather than distributional comparison against a real reference matrix
- When the input is bulk RNA-seq or non-transcriptomics count data outside the scRNA-seq/SRT
  scope of the benchmark
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_3308
tools:
- name: simpipe
  role: Standard pipeline that computes the 15 data properties (including LS, FZC,
    FZG, FGO, RMZ, RLZ) on real and simulated matrices and the distance metrics between
    them
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Provides the 49 simulation methods that generate the simulated expression
    matrices being characterized for zero inflation and library size
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
  source_content_hash: sha256:bfa7a84ba206141be5d131e96ea623ce22a5135916c56a655a65930a473592d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simpipe
  tools:
  - devtools
  - simmethods
  - simpipe
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Zero-inflation and library-size characterization

## Summary

Quantifies how well simulated scRNA-seq/SRT data reproduce zero-value properties (sparsity) and library-size properties of a real reference matrix, using the Simpipe accuracy module's data properties (LS, FZC, RLZ, FZG, FGO, RMZ, etc.) scored by 1-D distances and 2-D correlation-based metrics.

## When to use

Use when judging whether simulated single-cell or spatial transcriptomics matrices reproduce real-data sparsity (zero proportions) and library-size distributions, e.g., scoring FZC, FZG, FGO, LS, RLZ against a real reference.

## When NOT to use

- When the goal is functional evaluation (e.g., clustering or trajectory ground-truth fidelity) rather than accuracy on data properties such as sparsity and library size
- When only summary statistics (mean expression, variance) of the simulation are needed rather than distributional comparison against a real reference matrix
- When the input is bulk RNA-seq or non-transcriptomics count data outside the scRNA-seq/SRT scope of the benchmark

## Inputs

- Real reference scRNA-seq/SRT expression matrix (e.g., Simpipe vignette example matrix)
- Simulated expression matrices produced by Simmethods (one or more methods)
- R environment with devtools, simmethods, and simpipe installed

## Outputs

- metric_scores.csv with one row per data-property x metric per simulated dataset
- Per-property distance values (MAD, KS, MAE, RMSE, OV, BH for 1-D; multiKS and KDE-based distance for 2-D) for zero- and library-size properties

## How to apply

Install simmethods and simpipe from GitHub with devtools::install_github("duohongrui/simmethods") and devtools::install_github("duohongrui/simpipe"), then load the real reference expression matrix and the simulated counterpart matrices. For each zero- and library-size-related data property (LS, FZC, FZG, FGO, RMZ, RLZ among the 15 properties), compute the property vector on the real matrix and on every simulated matrix. For 1-D per-cell/per-gene property distributions, compute MAD, KS, MAE, RMSE, OV, and BH distances between real and simulated; for 2-D pairwise/correlation properties, compute the multiKS statistic and the KDE-based distance. Assemble one row per data-property x metric per simulated dataset into metric_scores.csv. The rationale is that no single parametric model captures sparsity and library size, so different methods capture different properties well — e.g., Simple cannot reproduce library size and per-cell zero proportions, while BASiCS does not characterize zero-value (sparsity) properties.

## Related tools

- **simpipe** (Standard pipeline that computes the 15 data properties (including LS, FZC, FZG, FGO, RMZ, RLZ) on real and simulated matrices and the distance metrics between them) — https://github.com/duohongrui/simpipe
- **simmethods** (Provides the 49 simulation methods that generate the simulated expression matrices being characterized for zero inflation and library size) — https://github.com/duohongrui/simmethods
- **devtools** (Used to install simmethods and simpipe from GitHub)

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- metric_scores.csv exists and contains rows for all 15 data properties crossed with the six 1-D metrics (MAD, KS, MAE, RMSE, OV, BH) and the two 2-D metrics (multiKS, KDE-based distance) for every simulated dataset
- All metric values are finite — no missing or infinite values, which the paper identifies as the main source of execution errors
- Zero-related properties (FZC, FZG, FGO, RMZ) and library-size properties (LS, RLZ) are each computed on both the real reference and every simulated matrix before distance calculation
- Methods known to fail sparsity (e.g., BASiCS) or library-size reproduction (e.g., Simple) should score poorly on the corresponding properties, providing a sanity check on metric directionality

## Limitations

- Execution errors are mainly caused by failed parameter estimations and the appearance of missing or infinite values in the metric calculations
- No simulation method performs best on all criteria, so a good property score on sparsity or library size alone does not imply overall best accuracy
- Modeling gene expression with a single parametric distribution may lose important information about data properties such as zero proportions

## Evidence

- [other] The accuracy module scores each method on individual data properties (e.g., cell–cell correlation (CCC) and fraction of cell outliers (FCO)) computed from real and simulated expression matrices: [evidence span withheld — anchor status: not_found]
- [methods] Install simmethods and simpipe from GitHub using devtools: "devtools::install_github("duohongrui/simmethods")"
- [abstract] Execution errors stem from missing/infinite values: "execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations"
- [discussion] Simple cannot reproduce library size and zero proportions: "which cannot accurately reproduce many properties of the real data, such as the library size of cells and the proportion of zeros per cell."
- [discussion] BASiCS does not characterize sparsity: "Nevertheless, BASiCS does not characterize the data properties associated with zero values to account for sparsity."
- [discussion] Single parametric distribution loses property information: "modeling gene expression data using a single parametric distribution may lead to the loss of important information about data properties."
