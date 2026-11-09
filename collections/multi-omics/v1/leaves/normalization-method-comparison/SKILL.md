---
name: normalization-method-comparison
description: Use when you must verify that a spatial multi-omics dimension-reduction/clustering
  pipeline's results (e.g., ARI of spatial domain detection) are stable across preprocessing
  choices such as normalization method and gene-set selection.
when_to_use_negative:
- When input is raw unnormalized count data intended for count-aware deep generative
  models — SMOPCA utilizes normalized data rather than raw expression measurements
  as inputs
- When the data lack spatial coordinates or a spatial kernel cannot be constructed
  (e.g., dissociated single-cell data without pseudo-spatial coordinates from UMAP)
- When the goal is method comparison rather than robustness assessment — this skill
  evaluates one method (SMOPCA) across preprocessing settings, not against competing
  methods like SpatialGlue or MEFISTO
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat V4
  role: Performs the three preprocessing variants (LogNormalize, SCTransform, VST)
    on the count matrix
  repo: https://github.com/satijalab/seurat
- name: SPARK
  role: Calls spatially variable genes at FDR threshold 0.05 using the Benjamini-Yekutieli
    procedure for the SVG gene-selection arm
- name: SRTsim
  role: Generates the simulated spatial multi-omics dataset from a DLPFC reference
    section, preserving expression characteristics and spatial patterns
- name: scikit-learn
  role: sklearn.cluster.KMeans clusters the SMOPCA low-dimensional vectors to produce
    labels for ARI evaluation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:7fdbb75a67af751e4a43839d3be28a2f778be09d7998182d09de6b8cd0575e5b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hemberg-lab/SC3
  repo_url: https://github.com/satijalab/seurat
  tools:
  - SPARK
  - SRTsim
  - Seurat V4
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# normalization-method-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A sensitivity-analysis protocol that benchmarks SMOPCA clustering performance across kernel parameters (Matern nu = 0.5/1.5/2.5), Seurat normalization methods (LogNormalize, SCTransform, VST), and gene-selection strategies (SPARK SVGs vs HVGs), demonstrating that spatial domain detection is robust to preprocessing choices.

## When to use

Use when you must verify that a spatial multi-omics dimension-reduction/clustering pipeline's results (e.g., ARI of spatial domain detection) are stable across preprocessing choices such as normalization method and gene-set selection.

## When NOT to use

- When input is raw unnormalized count data intended for count-aware deep generative models — SMOPCA utilizes normalized data rather than raw expression measurements as inputs
- When the data lack spatial coordinates or a spatial kernel cannot be constructed (e.g., dissociated single-cell data without pseudo-spatial coordinates from UMAP)
- When the goal is method comparison rather than robustness assessment — this skill evaluates one method (SMOPCA) across preprocessing settings, not against competing methods like SpatialGlue or MEFISTO

## Inputs

- Simulated spatial multi-omics dataset generated with SRTsim from a DLPFC reference section (count matrix + spatial coordinates)
- Raw gene expression count matrix per modality
- Simulated ground-truth spatial domain labels for ARI scoring

## Outputs

- Aggregated clustering performance table (robustness_results.csv) across nu, normalization, and gene-selection settings
- Comparison plots mirroring Additional file 1: Fig. S22-S31
- Per-combination ARI scores from KMeans clusters vs ground-truth spatial domains

## How to apply

Load the simulated spatial multi-omics dataset generated with SRTsim from a DLPFC reference section, which preserves expression characteristics and spatial patterns of the source data. Preprocess the count matrix three ways in Seurat: LogNormalize (library-size normalization, log transformation, scaling to zero mean and unit variance), SCTransform, and VST. For each normalization derive two gene sets: SPARK spatially variable genes called at FDR 0.05 with the Benjamini-Yekutieli procedure, and the top highly variable genes (HVGs; the paper used 5000 HVGs per DLPFC section for simulation input). Run SMOPCA on each of the 18 (normalization x gene-selection x kernel) combinations, constructing the spatial kernel matrix with a Matern kernel at nu = 0.5, 1.5, 2.5 and performing eigen decomposition on the kernel matrix for scalable computation. Cluster low-dimensional vectors with sklearn.cluster.KMeans, score against simulated ground-truth spatial domains (e.g., ARI), aggregate results, and write robustness_results.csv plus comparison plots mirroring Additional file 1: Fig. S22-S31.

## Related tools

- **Seurat V4** (Performs the three preprocessing variants (LogNormalize, SCTransform, VST) on the count matrix) — https://github.com/satijalab/seurat
- **SPARK** (Calls spatially variable genes at FDR threshold 0.05 using the Benjamini-Yekutieli procedure for the SVG gene-selection arm)
- **SRTsim** (Generates the simulated spatial multi-omics dataset from a DLPFC reference section, preserving expression characteristics and spatial patterns)
- **scikit-learn** (sklearn.cluster.KMeans clusters the SMOPCA low-dimensional vectors to produce labels for ARI evaluation)

## Examples

```
library(Seurat); so <- NormalizeData(so); so <- SCTransform(so); so <- ScaleData(so)
```

## Evaluation signals

- Aggregated table (robustness_results.csv) covers all 18 combinations: 3 normalizations x 2 gene selections x 3 Matern kernel nu values (0.5, 1.5, 2.5)
- ARI against simulated ground-truth spatial domains varies negligibly across nu, normalization, and gene-selection settings, consistent with the reported stability result (Additional file 1: Fig. S22-S31)
- SVG gene set derives from SPARK at FDR 0.05 with Benjamini-Yekutieli correction; HVG arm uses the top highly variable genes (5000 per DLPFC section in the simulation)
- Normalized matrices are scaled to zero mean and unit variance before dimension reduction
- KMeans is run on the SMOPCA low-dimensional vectors obtained via eigen decomposition of the Matern kernel matrix

## Limitations

- Robustness is demonstrated on SRTsim-simulated data from a DLPFC reference section; generalization to other tissues or platforms is not established here
- Only Matern kernels at nu = 0.5, 1.5, 2.5 are tested; SMOPCA's O(n^3) time and O(n^2) space complexity constrains scalability of the grid
- Lengthscale parameter learning may be challenging when data are very sparse and noisy or the spatial structure is very complicated, potentially requiring manual tuning
- SMOPCA does not model dropout events or the mean-variance relationship of raw counts and uses normalized data, which may reduce analytical power

## Evidence

- [readme] Seurat is the normalization toolkit: [evidence span withheld — anchor status: not_found]
