---
name: multi-omics-latent-factor-inference
description: Use when you have normalized expression/feature matrices for two or more
  omics modalities measured on the same spatial spots (e.g. spatial-CITE-seq, Stereo-CITE-seq)
  plus spot coordinates, and need joint dimension reduction that preserves spatial
  correlation for domain detection.
when_to_use_negative:
- 'Raw counts are required as input, e.g. count-aware deep generative models (totalVI,
  scVI): SMOPCA operates on normalized, log-transformed, scaled data and does not
  model the mean-variance relationship or dropout'
- Three or more modalities with a deep-learning method required — note SpaVAE and
  SpatialGlue handle at most two modalities without redesign, but this skill is the
  linear-Gaussian alternative, not a GNN/VAE replacement
- Very sparse/noisy data or highly complicated spatial structure where automatic lengthscale
  learning may fail and manual tuning would be required
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scikit-learn (sklearn.cluster.KMeans)
  role: K-means clustering on SMOPCA low-dimensional vectors to predict final spatial
    domain labels
- name: SpatialPCA
  role: State-of-the-art spatially aware dimension reduction baseline that SMOPCA
    extends from single-modality to multi-omics; builds on generalized probabilistic
    PCA with a spatial kernel matrix
  repo: https://github.com/shangll123/SpatialPCA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:1b116c9106038868a4f16fce15e47c10e2f8b4452a9bc2e5253a6a566a44af9e
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
  - SpatialPCA
  - scikit-learn (sklearn.cluster.KMeans)
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# multi-omics-latent-factor-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements SMOPCA, a spatial multi-omics principal component analysis method that infers a joint low-dimensional latent factor representation across multiple omics modalities while explicitly modeling spatial dependencies via a kernel-based shared covariance. Rationale: single-cell multi-omics methods assume independence among spots and fail to preserve neighboring similarity in tissue.

## When to use

Use when you have normalized expression/feature matrices for two or more omics modalities measured on the same spatial spots (e.g. spatial-CITE-seq, Stereo-CITE-seq) plus spot coordinates, and need joint dimension reduction that preserves spatial correlation for domain detection.

## When NOT to use

- Raw counts are required as input, e.g. count-aware deep generative models (totalVI, scVI): SMOPCA operates on normalized, log-transformed, scaled data and does not model the mean-variance relationship or dropout
- Three or more modalities with a deep-learning method required — note SpaVAE and SpatialGlue handle at most two modalities without redesign, but this skill is the linear-Gaussian alternative, not a GNN/VAE replacement
- Very sparse/noisy data or highly complicated spatial structure where automatic lengthscale learning may fail and manual tuning would be required

## Inputs

- Normalized expression/feature matrices for two or more modalities (e.g. gene expression, protein ADT, gene-activity matrix collapsed from ATAC peaks) on matched spots
- Spatial coordinates (x, y) for each spot
- Optionally, UMAP-derived pseudo-spatial coordinates for single-cell multi-omics data (top 50 PCs)

## Outputs

- Joint latent factor matrix Z of shape (n_spots, n_factors) with finite values (e.g. saved to latent_factors.csv)
- Learned kernel lengthscale parameter
- Modality-specific loading matrices W_k
- Downstream K-means spatial domain cluster labels

## How to apply

Generate normalized input matrices per modality (library-size normalization, log transformation, scaling to zero mean and unit variance) together with per-spot spatial coordinates. Build a shared spatial covariance by evaluating a Matern nu=3/2 kernel on the spot coordinates and performing eigen decomposition on the kernel matrix for scalable computation. Estimate modality-specific parameters (sigma_k, W_k) via closed-form maximum marginal likelihood updates, iterating across modalities with the latent-factor covariance scale gamma updated between modalities, and learn the kernel lengthscale by maximizing the model marginal likelihood. Compute the posterior mean of the latent factors Z as the integrated low-dimensional representation, then optionally apply sklearn.cluster.KMeans on the low-dimensional vectors for spatial domain labels.

## Related tools

- **scikit-learn (sklearn.cluster.KMeans)** (K-means clustering on SMOPCA low-dimensional vectors to predict final spatial domain labels)
- **SpatialPCA** (State-of-the-art spatially aware dimension reduction baseline that SMOPCA extends from single-modality to multi-omics; builds on generalized probabilistic PCA with a spatial kernel matrix) — https://github.com/shangll123/SpatialPCA

## Examples

```
from sklearn.gaussian_process.kernels import Matern; import numpy as np; K = Matern(nu=1.5, length_scale=l)(np.asarray(coords)); evals, evecs = np.linalg.eigh(K)
```

## Evaluation signals

- Z has shape (n_spots, n_factors) and all values are finite (no NaN/Inf)
- Latent factors preserve neighboring similarity: spatial correlation structure of the original spots is maintained (check correlation of Z against spatial adjacency)
- Learned lengthscale converges to a finite positive value via marginal likelihood maximization rather than requiring manual tuning on well-structured data
- Closed-form MLE updates of (sigma_k, W_k) decrease or stabilize the model marginal likelihood across iterations and modalities
- Downstream K-means on Z improves spatial domain detection (e.g. ARI against manual DLPFC layer annotations) versus single-modal or per-modality dimension reduction

## Limitations

- Current implementation has O(n^3) time and O(n^2) space complexity; authors aim to reduce both in future work
- Does not model dropout events or the inherent mean-variance relationship of raw counts, and uses normalized data rather than raw expression measurements, potentially reducing analytical power
- Shared covariance assumption (Sigma_1 = ... = Sigma_d) is imposed for computational convenience; relaxing it requires orthogonal-constrained (Stiefel manifold) numerical optimization with no closed-form solution
- Learning the lengthscale parameter may be challenging when data are very sparse and noisy or the spatial structure is very complicated, requiring manual user tuning

## Evidence

- [other] Build the shared spatial covariance by evaluating a Matern nu=3/2 kernel on the spot coordinates and performing eigen decomposition on the kernel matrix for scalable computation.: [evidence span withheld — anchor status: not_found]
