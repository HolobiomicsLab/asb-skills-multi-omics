---
name: matern-kernel-sensitivity-analysis
description: Use when you need to test whether a spatially aware dimension reduction
  and clustering pipeline (e.g., SMOPCA on simulated spatial multi-omics data) is
  robust to kernel parameters, normalization methods, and gene-selection strategies.
when_to_use_negative:
- Goal is benchmarking SMOPCA against competing methods (SpatialGlue, MEFISTO, TotalVI,
  etc.) rather than testing SMOPCA's internal robustness to its own settings
- Input data lack ground-truth spatial domain labels, so ARI scoring of the clustering
  against simulated domains is impossible
- Analysis targets non-spatial single-cell multi-omics without coordinates and you
  are not using the UMAP pseudo-spatial-coordinate extension described in the paper
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SRTsim
  role: Generates the simulated spatial multi-omics dataset from a DLPFC reference
    section, preserving expression characteristics and spatial patterns used as sensitivity-analysis
    input
- name: SPARK
  role: Calls spatially variable genes (SVGs) at an FDR threshold of 0.05 with the
    Benjamini-Yekutieli procedure for the SVG gene-selection arm
- name: Seurat V4
  role: Performs the three preprocessing variants (LogNormalize, SCTransform, VST)
    of the count matrix and derives HVG gene sets
  repo: https://github.com/satijalab/seurat
- name: scikit-learn
  role: Supplies sklearn.cluster.KMeans used to cluster the low-dimensional vectors
    from the eigen-decomposed kernel matrix before ARI scoring
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:6dc90bec418fc722628def25b224484ac237594f2038e1bdc2ea451953d08487
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# matern-kernel-sensitivity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces SMOPCA's robustness analysis (Additional file 1: Fig. S22-S31) by running the spatial multi-omics PCA pipeline across Matern kernels (nu = 0.5, 1.5, 2.5), three Seurat normalization methods (LogNormalize, SCTransform, VST), and SVG-vs-HVG gene selection, scoring each clustering with ARI against simulated ground-truth spatial domains.

## When to use

Use when you need to test whether a spatially aware dimension reduction and clustering pipeline (e.g., SMOPCA on simulated spatial multi-omics data) is robust to kernel parameters, normalization methods, and gene-selection strategies.

## When NOT to use

- Goal is benchmarking SMOPCA against competing methods (SpatialGlue, MEFISTO, TotalVI, etc.) rather than testing SMOPCA's internal robustness to its own settings
- Input data lack ground-truth spatial domain labels, so ARI scoring of the clustering against simulated domains is impossible
- Analysis targets non-spatial single-cell multi-omics without coordinates and you are not using the UMAP pseudo-spatial-coordinate extension described in the paper

## Inputs

- SRTsim-simulated spatial multi-omics count matrix generated from a DLPFC reference section (with spatial coordinates)
- Ground-truth spatial domain labels for the simulated spots
- Seurat-normalized expression matrices (LogNormalize, SCTransform, VST variants)
- Gene sets: SPARK SVGs (FDR < 0.05, Benjamini-Yekutieli) and top HVGs

## Outputs

- robustness_results.csv with ARI scores for all 18 (normalization x gene-selection x Matern nu) combinations
- Comparison plots mirroring Additional file 1: Fig. S22-S31
- Per-combination KMeans cluster label vectors on the low-dimensional representations

## How to apply

Load the spatial multi-omics dataset simulated with SRTsim from a DLPFC reference section, which preserves expression characteristics and spatial patterns of the source data. Preprocess the count matrix three ways in Seurat: LogNormalize (library-size normalization, log transformation, scaling to zero mean and unit variance), SCTransform, and VST. For each normalization derive two gene sets: SPARK spatially variable genes called at an FDR threshold of 0.05 with the Benjamini-Yekutieli procedure, and the top highly variable genes (HVGs). For each of the 18 (normalization x gene-selection x kernel) combinations, construct the spatial kernel matrix with a Matern kernel at nu = 0.5, 1.5, and 2.5, perform eigen decomposition on the kernel matrix to obtain the low-dimensional representation, then cluster with sklearn.cluster.KMeans. Score each clustering against the simulated ground-truth spatial domains (e.g., ARI), aggregate across settings, render comparison plots mirroring Additional file 1: Fig. S22-S31, and write the aggregated table to robustness_results.csv.

## Related tools

- **SRTsim** (Generates the simulated spatial multi-omics dataset from a DLPFC reference section, preserving expression characteristics and spatial patterns used as sensitivity-analysis input)
- **SPARK** (Calls spatially variable genes (SVGs) at an FDR threshold of 0.05 with the Benjamini-Yekutieli procedure for the SVG gene-selection arm)
- **Seurat V4** (Performs the three preprocessing variants (LogNormalize, SCTransform, VST) of the count matrix and derives HVG gene sets) — https://github.com/satijalab/seurat
- **scikit-learn** (Supplies sklearn.cluster.KMeans used to cluster the low-dimensional vectors from the eigen-decomposed kernel matrix before ARI scoring)

## Examples

```
from sklearn.cluster import KMeans; from sklearn.metrics import adjusted_rand_score; ari = adjusted_rand_score(KMeans(n_clusters=7).fit_predict(Z), truth)
```

## Evaluation signals

- All 18 combinations (3 normalizations x 2 gene sets x 3 Matern nu values = 0.5, 1.5, 2.5) yield a valid ARI score with no failed runs
- robustness_results.csv is produced and ARI varies negligibly across nu, normalization, and gene-selection settings, consistent with Additional file 1: Fig. S22-S31
- SVG gene sets are the output of SPARK at FDR 0.05 using the Benjamini-Yekutieli procedure, not an arbitrary significance cutoff
- Low-dimensional representations are obtained by eigen decomposition of the Matern kernel matrix, followed by sklearn.cluster.KMeans clustering (not graph-based or model-based clustering)
- Comparison plots reproduce the layout and comparison structure of Additional file 1: Fig. S22-S31

## Limitations

- The robustness conclusion is established on SRTsim-simulated data generated from a DLPFC reference section, not on real tissue with experimentally annotated domains
- Learning the kernel lengthscale parameter may be challenging when the data are very sparse and noisy or the spatial structure is very complicated; manual user tuning may be required
- SMOPCA uses normalized data rather than raw counts and has O(n^3) time / O(n^2) space complexity, which constrains dataset size for the 18-combination sweep

## Evidence

- [supplementary] Robustness finding: [evidence span withheld — anchor status: not_found]
- [readme] Kernel matrix role (related method context): [evidence span withheld — anchor status: external_doc]
