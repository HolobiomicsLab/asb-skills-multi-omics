---
name: simulation-replicate-aggregation
description: Use when comparing clustering methods on single-cell multi-omics data
  (e.g. CITE-seq or SMAGE-seq) that lack true spatial coordinates, and you need replicated
  simulated-2D-grid benchmarks with AMI/NMI/ARI summaries.
when_to_use_negative:
- Data already have genuine spatial coordinates (e.g. DLPFC, spatial-CITE-seq human
  tonsil GSE213264, Stereo-CITE-seq mouse thymus) — use real spatial domain detection
  evaluation instead of simulated grids
- Datasets lack ground-truth cell-type labels, making AMI/NMI/ARI computation against
  a reference impossible
- The goal is to evaluate a method's behavior on true tissue spatial structure rather
  than controlled simulated coordinates
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SMOPCA
  role: 'Method under test: eigen decomposition on the kernel matrix, iterative MLE
    of sigma_k and W_k, posterior mean of Z as integrated representation before K-means'
- name: SpatialPCA
  role: Competing spatially aware baseline, applied to concatenated modalities on
    the same simulated coordinates
  repo: https://github.com/shangll123/SpatialPCA
- name: scMDC
  role: Competing multimodal deep-clustering baseline run on the same preprocessed
    data and simulated coordinates
  repo: https://github.com/xianglin226/scMDC
- name: Seurat V4
  role: Single-modality competing baseline for preprocessing and clustering comparison
  repo: https://github.com/satijalab/seurat
- name: scikit-learn
  role: Provides sklearn.cluster.KMeans used to predict final clustering labels on
    the low-dimensional vectors
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:dc2c3ce67672e28f7a6eeb3eb055416339c1e476f078ef9cf2218f248d53bd19
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hemberg-lab/SC3
  repo_url: https://github.com/shangll123/SpatialPCA
  tools:
  - SMOPCA
  - Seurat V4
  - SpatialPCA
  - scMDC
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# simulation-replicate-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking skill from the SMOPCA paper (Genome Biology, 2025): simulate spatial coordinates for single-cell multi-omics data, run a dimension-reduction/clustering pipeline plus baselines across repeated simulations, and aggregate AMI/NMI/ARI into per-dataset metric distributions.

## When to use

Use when comparing clustering methods on single-cell multi-omics data (e.g. CITE-seq or SMAGE-seq) that lack true spatial coordinates, and you need replicated simulated-2D-grid benchmarks with AMI/NMI/ARI summaries.

## When NOT to use

- Data already have genuine spatial coordinates (e.g. DLPFC, spatial-CITE-seq human tonsil GSE213264, Stereo-CITE-seq mouse thymus) — use real spatial domain detection evaluation instead of simulated grids
- Datasets lack ground-truth cell-type labels, making AMI/NMI/ARI computation against a reference impossible
- The goal is to evaluate a method's behavior on true tissue spatial structure rather than controlled simulated coordinates

## Inputs

- Seven preprocessed single-cell multi-omics count matrices (CITE-seq: PBMC, SLN111D1, SLN111D2, SLN208D1, SLN208D2; SMAGE-seq: PBMC3K, PBMC10K)
- Ground-truth cell-type labels for each dataset
- Simulated 2D grid spatial coordinates per dataset (cells allocated row by row and column by column)
- Low-dimensional latent representations from SMOPCA, SpatialPCA, scMDC, PCA, and single-modality baselines

## Outputs

- simulated_spatial_clustering_metrics.csv with per-dataset AMI, NMI, and ARI distributions across 10 replicates per method
- Per-replicate cluster label assignments from K-means on the low-dimensional vectors

## How to apply

Preprocess each count matrix by library-size normalization, log transformation, and scaling to zero mean and unit variance, after filtering genes/proteins with zero counts. Simulate spatial locations by allocating cells onto a 2D grid row by row and column by column, then run the method under test (SMOPCA: eigen decomposition on the kernel matrix, iterative estimation of modality-specific sigma_k and W_k, posterior mean of Z as the integrated representation) and the competing methods (SpatialPCA on concatenated modalities, scMDC, PCA + K-means, single-modality baselines) on the same simulated coordinates. Predict cluster labels with sklearn.cluster.KMeans on the low-dimensional vectors. Repeat the simulation and clustering for 10 replicates per dataset (five CITE-seq: PBMC, SLN111D1, SLN111D2, SLN208D1, SLN208D2; two SMAGE-seq: PBMC3K, PBMC10K). Compute AMI, NMI, and ARI against ground-truth cell-type labels for every method and replicate, and save the per-dataset metric distributions to simulated_spatial_clustering_metrics.csv.

## Related tools

- **SMOPCA** (Method under test: eigen decomposition on the kernel matrix, iterative MLE of sigma_k and W_k, posterior mean of Z as integrated representation before K-means)
- **SpatialPCA** (Competing spatially aware baseline, applied to concatenated modalities on the same simulated coordinates) — https://github.com/shangll123/SpatialPCA
- **scMDC** (Competing multimodal deep-clustering baseline run on the same preprocessed data and simulated coordinates) — https://github.com/xianglin226/scMDC
- **Seurat V4** (Single-modality competing baseline for preprocessing and clustering comparison) — https://github.com/satijalab/seurat
- **scikit-learn** (Provides sklearn.cluster.KMeans used to predict final clustering labels on the low-dimensional vectors)

## Examples

```
from sklearn.cluster import KMeans; labels = KMeans(n_clusters=n_types, random_state=rep).fit_predict(Z_posterior_mean)
```

## Evaluation signals

- Exactly 10 replicates of simulation and clustering are run per dataset (7 datasets x 10 replicates per method)
- AMI, NMI, and ARI are computed against ground-truth cell-type labels for every method and replicate, not just the mean over replicates
- SMOPCA should outperform SpatialPCA (on concatenated modalities) across all three metrics, and beat scMDC, PCA + K-means, and single-modality baselines
- All methods must be evaluated on identical simulated coordinates and identically preprocessed data (library-size normalized, log-transformed, scaled to zero mean and unit variance)
- Output CSV contains per-dataset metric distributions, enabling downstream visualization of replicate variability

## Limitations

- Simulated 2D grid coordinates do not reflect real tissue spatial structure, so results demonstrate relative robustness rather than true spatial domain detection performance
- SMOPCA's current implementation has O(n^3) time and O(n^2) space complexity, which constrains the scale of each simulation replicate
- Learning the lengthscale parameter may be challenging when data are sparse and noisy or the spatial structure is complicated, potentially requiring manual tuning

## Evidence
