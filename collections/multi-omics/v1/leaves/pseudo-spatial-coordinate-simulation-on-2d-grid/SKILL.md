---
name: pseudo-spatial-coordinate-simulation-on-2d-grid
description: Use when benchmarking spatially aware multi-omics dimension reduction/clustering
  methods (e.g. SMOPCA vs SpatialPCA, scMDC) on single-cell multi-omics datasets such
  as CITE-seq or SMAGE-seq count matrices that lack real spatial coordinates.
when_to_use_negative:
- The data already carry real spatial coordinates (e.g. DLPFC spatial transcriptomics,
  spatial-CITE-seq tonsil, Stereo-CITE-seq thymus) — use the actual locations instead
  of a simulated grid
- The goal is simulation that preserves realistic spatial expression patterns — the
  paper used SRTsim on 5000 HVGs per DLPFC section for that, not 2D grid allocation
- Raw-count-aware deep methods are required and data are very sparse/noisy — SMOPCA's
  grid-simulation benchmark uses normalized (non-raw) data and does not model dropout
  or the mean–variance relationship
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SMOPCA
  role: Spatially aware multi-omics PCA run on the simulated 2D grid coordinates to
    produce the integrated latent representation being benchmarked
- name: SpatialPCA
  role: Competing spatially aware dimension reduction method applied to concatenated
    modalities on the same simulated coordinates
  repo: https://github.com/shangll123/SpatialPCA
- name: scMDC
  role: Competing multimodal deep clustering method run on the same simulated datasets
  repo: https://github.com/xianglin226/scMDC
- name: scikit-learn
  role: Provides sklearn.cluster.KMeans used to predict final cluster labels from
    SMOPCA's low-dimensional vectors
- name: Seurat V4
  role: Comparison method/baseline cited in the benchmarking context
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:6722d49d076abd5209c76c560b9aaa0b90336560a8bf2e18225e3260611d8ede
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# Pseudo-spatial coordinate simulation on 2D grid

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking technique from the SMOPCA paper that allocates cells from single-cell multi-omics count matrices (CITE-seq, SMAGE-seq) onto a simulated 2D grid, row by row and column by column, so that spatially aware methods like SMOPCA and SpatialPCA can be run and compared on data that has no real spatial locations.

## When to use

Use when benchmarking spatially aware multi-omics dimension reduction/clustering methods (e.g. SMOPCA vs SpatialPCA, scMDC) on single-cell multi-omics datasets such as CITE-seq or SMAGE-seq count matrices that lack real spatial coordinates.

## When NOT to use

- The data already carry real spatial coordinates (e.g. DLPFC spatial transcriptomics, spatial-CITE-seq tonsil, Stereo-CITE-seq thymus) — use the actual locations instead of a simulated grid
- The goal is simulation that preserves realistic spatial expression patterns — the paper used SRTsim on 5000 HVGs per DLPFC section for that, not 2D grid allocation
- Raw-count-aware deep methods are required and data are very sparse/noisy — SMOPCA's grid-simulation benchmark uses normalized (non-raw) data and does not model dropout or the mean–variance relationship

## Inputs

- Single-cell multi-omics raw count matrices: five CITE-seq datasets (PBMC 3762 cells/33,538 genes/49 proteins; SLN111D1 8853 cells; SLN111D2 6949 cells; SLN208D1 8371 cells; SLN208D2 6678 cells)
- Two SMAGE-seq count matrices (PBMC3K ~3000 cells, PBMC10K ~10,000 cells)
- Ground-truth cell-type labels for AMI/NMI/ARI computation

## Outputs

- Simulated 2D spatial location grids for each dataset
- Per-dataset AMI, NMI, ARI metric distributions over 10 replicates for SMOPCA and competing methods (saved to e.g. simulated_spatial_clustering_metrics.csv)
- KMeans cluster label predictions per method and replicate

## How to apply

Load the seven single-cell multi-omics count matrices (CITE-seq: PBMC, SLN111D1, SLN111D2, SLN208D1, SLN208D2; SMAGE-seq: PBMC3K, PBMC10K), filter zero-count features, then preprocess by library-size normalization, log transformation, and scaling to zero mean and unit variance. Allocate the cells to 2D grid locations row by row and column by column to fabricate pseudo-spatial coordinates. Run SMOPCA on each simulated dataset — eigen decomposition of the kernel matrix, iterative estimation of modality-specific parameters (sigma_k, W_k), posterior mean of latent Z as the integrated representation — and predict cluster labels with sklearn.cluster.KMeans on the low-dimensional vectors; run SpatialPCA (on concatenated modalities), scMDC, and PCA+K-means baselines on the same simulated coordinates. Repeat for 10 replicates per dataset and compute AMI, NMI, and ARI against the ground-truth cell-type labels, saving per-dataset metric distributions (e.g. simulated_spatial_clustering_metrics.csv).

## Related tools

- **SMOPCA** (Spatially aware multi-omics PCA run on the simulated 2D grid coordinates to produce the integrated latent representation being benchmarked)
- **SpatialPCA** (Competing spatially aware dimension reduction method applied to concatenated modalities on the same simulated coordinates) — https://github.com/shangll123/SpatialPCA
- **scMDC** (Competing multimodal deep clustering method run on the same simulated datasets) — https://github.com/xianglin226/scMDC
- **scikit-learn** (Provides sklearn.cluster.KMeans used to predict final cluster labels from SMOPCA's low-dimensional vectors)
- **Seurat V4** (Comparison method/baseline cited in the benchmarking context) — https://github.com/satijalab/seurat

## Examples

```
import numpy as np; from sklearn.cluster import KMeans; coords = np.array([(r, c) for r in range(nrows) for c in range(ncols)]); labels = KMeans(n_clusters=n_types, random_state=0).fit_predict(Z)
```

## Evaluation signals

- Every cell in the dataset is allocated to exactly one 2D grid location, filled row by row and column by column, with no duplicated or missing positions
- Preprocessing invariants: library-size normalization followed by log transformation and scaling to zero mean and unit variance; zero-count genes/proteins filtered out first
- Each dataset/method combination is repeated over 10 replicates and all three metrics (AMI, NMI, ARI) are computed against ground-truth cell-type labels
- Expected result: SMOPCA outperforms SpatialPCA (on concatenated modalities) on all three metrics and beats scMDC and PCA+K-means baselines across the five CITE-seq and two SMAGE-seq datasets

## Limitations

- Simulated 2D grid coordinates impose an artificial spatial structure that does not correspond to real tissue topology; for spatially realistic simulations the authors instead used SRTsim on DLPFC data
- SMOPCA (the method under benchmark) does not model dropout events or the mean–variance relationship of raw counts and takes normalized data rather than raw measurements
- SMOPCA's current implementation has O(n^3) time and O(n^2) space complexity, and the lengthscale parameter may be hard to learn on very sparse/noisy data, requiring manual tuning

## Evidence
