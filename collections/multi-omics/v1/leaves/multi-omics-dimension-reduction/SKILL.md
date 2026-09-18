---
name: multi-omics-dimension-reduction
description: Use when you have paired multi-omics count matrices (e.g. gene expression
  + protein ADT, or ATAC gene activity + mRNA) measured on the same cells/spots and
  need an integrated low-dimensional representation for clustering, with or without
  spatial coordinates.
when_to_use_negative:
- Data are raw counts where dropout events and the mean-variance relationship matter
  — SMOPCA takes normalized data as input and does not model count distributions,
  unlike count-aware deep generative methods (TotalVI, PeakVI)
- Modalities are unpaired (measured on different cells), since the model integrates
  modalities measured on the same set of cells/spots
- Very sparse/noisy data with complicated spatial structure where the kernel lengthscale
  parameter cannot be learned reliably and manual tuning would dominate results
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: SMOPCA
  role: 'Core spatial multi-omics PCA model: eigen decomposition of kernel matrix,
    iterative MLE of (σ_k, W_k), posterior mean of Z as integrated representation'
- name: SpatialPCA
  role: Spatially aware dimension reduction baseline applied to concatenated modalities
    or single modality for comparison
  repo: https://github.com/shangll123/SpatialPCA
- name: scMDC
  role: Deep multimodal clustering baseline for single-cell multi-omics data
  repo: https://github.com/xianglin226/scMDC
- name: Seurat V4
  role: Used to generate 2D UMAP pseudo-spatial coordinates from the top 50 PCs for
    single-cell multi-omics inputs
  repo: https://github.com/satijalab/seurat
- name: scikit-learn KMeans
  role: K-means clustering (sklearn.cluster.KMeans) on the low-dimensional vectors
    to predict final cluster labels
- name: SPARK
  role: Selects spatially variable genes (FDR 0.05, Benjamini-Yekutieli) as input
    features for the spatial applications
- name: BREM-SC
  role: Single-cell multi-omics clustering baseline
  repo: https://github.com/tarot0410/BREMSC
- name: SpatialGlue
  role: Graph neural network baseline; SpatialGlue_3M extension handles three modalities
  repo: https://github.com/JinmiaoChenLab/SpatialGlue_3M
- name: MEFISTO
  role: Multi-omics factor analysis baseline that natively handles 3+ modalities
  repo: https://biofam.github.io/MOFA2/MEFISTO
- name: TotalVI
  role: Probabilistic joint RNA+protein modeling baseline
  repo: https://scvi-tools.org
merged_aliases:
- spatial-multi-omics-dimension-reduction
merged_alias_records:
- alias: spatial-multi-omics-dimension-reduction
  slug: spatial-multi-omics-dimension-reduction
  jaccard_score: 0.8
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_006
  - task_007
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:ec7c0d415a28c0051372d36ec91ed36dfa43a0d400884926fe9f931c7231a3db
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
  - BREM-SC
  - MEFISTO
  - SMOPCA
  - SPARK
  - SRTsim
  - Seurat V4
  - SpaVAE
  - SpatialGlue
  - SpatialGlue_3M
  - SpatialPCA
  - TotalVI
  - scMDC
  - scikit-learn
  - scikit-learn KMeans
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# multi-omics-dimension-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Jointly reduce the dimensionality of multi-omics data (CITE-seq, SMAGE-seq, spatial-CITE-seq, MISAR-seq ATAC+mRNA) by fitting a spatially aware probabilistic PCA model (SMOPCA) that preserves spatial dependencies and integrates modality-specific information into a single latent representation. The posterior mean of the latent Z replaces per-modality PCA as input for K-means clustering and spatial domain detection.

## When to use

Use when you have paired multi-omics count matrices (e.g. gene expression + protein ADT, or ATAC gene activity + mRNA) measured on the same cells/spots and need an integrated low-dimensional representation for clustering, with or without spatial coordinates.

## When NOT to use

- Data are raw counts where dropout events and the mean-variance relationship matter — SMOPCA takes normalized data as input and does not model count distributions, unlike count-aware deep generative methods (TotalVI, PeakVI)
- Modalities are unpaired (measured on different cells), since the model integrates modalities measured on the same set of cells/spots
- Very sparse/noisy data with complicated spatial structure where the kernel lengthscale parameter cannot be learned reliably and manual tuning would dominate results

## Inputs

- Paired multi-omics count matrices measured on the same cells/spots (e.g. CITE-seq gene expression + 49 ADT proteins; spatial-CITE-seq 28,417 genes + 283 proteins; MISAR-seq gene activity + mRNA)
- Spatial coordinates for each spot/cell, or simulated 2D grid / UMAP-derived pseudo-coordinates for single-cell data
- Preprocessed matrices: library-size normalized, log-transformed, scaled to zero mean and unit variance
- Optionally, SPARK-selected spatially variable genes (FDR 0.05, BY procedure) per modality

## Outputs

- Posterior mean of the latent Z: an integrated low-dimensional representation per cell/spot
- K-means cluster labels / spatial domain assignments (sklearn.cluster.KMeans) computed on the low-dimensional vectors
- Benchmarking metrics such as AMI, NMI, ARI against ground-truth labels, Moran's I, LISI, and McFadden pseudo R2 when evaluated

## How to apply

Preprocess each modality by library-size normalization, log transformation, and scaling to zero mean and unit variance; filter genes/proteins with zero counts and, for spatial data, select spatially variable genes with SPARK (FDR 0.05, Benjamini-Yekutieli procedure). For single-cell data, supply pseudo-spatial coordinates: either allocate cells row by row and column by column onto a 2D grid, or generate 2D UMAP coordinates from the top 50 PCs in Seurat. Fit SMOPCA by performing eigen decomposition on the kernel matrix Σ for scalability, iteratively estimating modality-specific parameters (σ_k, W_k) with γ updated across modalities, and compute the posterior mean of Z as the integrated low-dimensional representation; spatial correlation is modeled explicitly via a kernel matrix, which distinguishes this from independence-assuming single-cell methods. Finally run sklearn.cluster.KMeans on the low-dimensional vectors to predict cluster labels or spatial domains.

## Related tools

- **SMOPCA** (Core spatial multi-omics PCA model: eigen decomposition of kernel matrix, iterative MLE of (σ_k, W_k), posterior mean of Z as integrated representation)
- **SpatialPCA** (Spatially aware dimension reduction baseline applied to concatenated modalities or single modality for comparison) — https://github.com/shangll123/SpatialPCA
- **scMDC** (Deep multimodal clustering baseline for single-cell multi-omics data) — https://github.com/xianglin226/scMDC
- **Seurat V4** (Used to generate 2D UMAP pseudo-spatial coordinates from the top 50 PCs for single-cell multi-omics inputs) — https://github.com/satijalab/seurat
- **scikit-learn KMeans** (K-means clustering (sklearn.cluster.KMeans) on the low-dimensional vectors to predict final cluster labels)
- **SPARK** (Selects spatially variable genes (FDR 0.05, Benjamini-Yekutieli) as input features for the spatial applications)
- **BREM-SC** (Single-cell multi-omics clustering baseline) — https://github.com/tarot0410/BREMSC
- **SpatialGlue** (Graph neural network baseline; SpatialGlue_3M extension handles three modalities) — https://github.com/JinmiaoChenLab/SpatialGlue_3M
- **MEFISTO** (Multi-omics factor analysis baseline that natively handles 3+ modalities) — https://biofam.github.io/MOFA2/MEFISTO
- **TotalVI** (Probabilistic joint RNA+protein modeling baseline) — https://scvi-tools.org

## Examples

```
from sklearn.cluster import KMeans; labels = KMeans(n_clusters=7, random_state=0).fit_predict(smopca_posterior_mean_Z)
```

## Evaluation signals

- Clustering metrics (AMI, NMI, ARI) computed against ground-truth cell-type or spatial domain labels should exceed competing baselines (SpatialPCA, scMDC, PCA+K-means, Seurat, TotalVI)
- On spatial data, Moran's I of the clusters should be the highest and LISI the lowest relative to baselines, indicating spatially homogeneous, non-mixed clusters
- McFadden-adjusted pseudo R2 of the latent representation should be comparable to or better than competing dimension reduction methods (chromVAR, cisTopic, LSA, PeakVI, SCALE)
- Per-modality parameter estimation converges: σ_k and W_k iterates stabilize across modalities with γ updated across modalities
- Cluster annotations verified against marker expression (e.g. CD3/CD4/CD49a up in T-cell zone cluster; CD21/CD23/IgM up in germinal-center clusters)

## Limitations

- Current implementation has O(n^3) time and O(n^2) space complexity; the authors aim to reduce both in future work
- The method does not model dropout events or the mean-variance relationship of raw counts, and uses normalized data rather than raw counts, which may reduce analytical power
- The shared covariance assumption (Σ1=...=Σd) is imposed for computational convenience; relaxing it requires orthogonal-constrained (Stiefel manifold) numerical optimization, deferred to future work
- Learning the lengthscale parameter may be challenging when data are sparse/noisy or spatial structure is complicated, requiring manual user tuning

## Evidence

- [readme] SpatialPCA README: kernel matrix models spatial correlation: [evidence span withheld — anchor status: external_doc]
