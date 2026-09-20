---
name: spatial-kernel-covariance-construction
description: Use when you have normalized multi-modal omics matrices plus per-spot
  spatial coordinates and need a shared covariance that encodes spatial correlation
  for a probabilistic PCA-style latent factor model.
when_to_use_negative:
- Data have no spatial coordinates and no defensible pseudo-spatial embedding — the
  kernel covariance cannot be built and independent-spot multi-omics methods apply
  instead
- Raw counts with dropout and mean-variance structure must be modeled directly (e.g.
  count-aware deep generative models like scVI/TotalVI); SMOPCA-style kernel covariance
  operates on normalized data only
- Per-modality spatial covariances genuinely differ (Sigma_1 != ... != Sigma_d); the
  shared-covariance closed-form solution does not exist and orthogonal-constrained
  (Stiefel manifold) numerical optimization would be required
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SpatialPCA
  role: State-of-the-art spatially aware dimension reduction method whose kernel-matrix
    modeling of spatial correlation SMOPCA's covariance construction builds upon and
    extends to multi-omics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:9318f548cb0ac776e593c82e00e9f9a3da6d822a5a029a8f875e047573653621
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hemberg-lab/SC3
  tools:
  - SpatialPCA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# spatial-kernel-covariance-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs the shared spatial covariance of SMOPCA by evaluating a Matern nu=3/2 kernel on spot coordinates and performing eigen decomposition on the resulting kernel matrix, so that latent factors preserve spatial dependencies across tissue locations.

## When to use

Use when you have normalized multi-modal omics matrices plus per-spot spatial coordinates and need a shared covariance that encodes spatial correlation for a probabilistic PCA-style latent factor model.

## When NOT to use

- Data have no spatial coordinates and no defensible pseudo-spatial embedding — the kernel covariance cannot be built and independent-spot multi-omics methods apply instead
- Raw counts with dropout and mean-variance structure must be modeled directly (e.g. count-aware deep generative models like scVI/TotalVI); SMOPCA-style kernel covariance operates on normalized data only
- Per-modality spatial covariances genuinely differ (Sigma_1 != ... != Sigma_d); the shared-covariance closed-form solution does not exist and orthogonal-constrained (Stiefel manifold) numerical optimization would be required

## Inputs

- Spatial coordinates for each spot (2D x/y positions from spatial transcriptomics, or pseudo-coordinates from 2D UMAP for single-cell data)
- Normalized expression matrices for two or more modalities (library-size normalized, log-transformed, scaled to zero mean and unit variance)
- Initial kernel lengthscale value

## Outputs

- Shared spatial covariance kernel matrix Sigma over spots
- Eigen decomposition of the kernel matrix (eigenvalues and eigenvectors)
- Learned kernel lengthscale parameter
- Posterior mean of latent factors Z of shape (n_spots, n_factors) used as the integrated low-dimensional representation

## How to apply

Evaluate a Matern nu=3/2 kernel on the spot coordinates to build the shared spatial covariance matrix Sigma, which SMOPCA imposes as Sigma_1=...=Sigma_d across all modalities for computational tractability. Perform eigen decomposition on the kernel matrix Sigma for scalable computation, then feed its structure into the iterative closed-form maximum marginal likelihood updates of modality-specific parameters (sigma_k, W_k) with gamma updated between modalities. Learn the kernel lengthscale automatically by maximizing the model marginal likelihood; fall back to manual tuning when data are sparse/noisy or the spatial structure is complicated. The kernel-covariance idea follows SpatialPCA, which uses a kernel matrix to explicitly model the spatial correlation structure across tissue locations, extended here to multiple omics modalities simultaneously.

## Related tools

- **SpatialPCA** (State-of-the-art spatially aware dimension reduction method whose kernel-matrix modeling of spatial correlation SMOPCA's covariance construction builds upon and extends to multi-omics)

## Examples

```
from sklearn.gaussian_process.kernels import Matern; import numpy as np; K = Matern(length_scale=l, nu=1.5)(coords); vals, vecs = np.linalg.eigh(K)
```

## Evaluation signals

- Kernel matrix Sigma is symmetric and positive semi-definite; its eigen decomposition yields finite, non-negative eigenvalues
- Learned lengthscale is strictly positive and increases the model marginal likelihood when optimized
- Posterior mean Z has shape (n_spots, n_factors) with all finite values
- The learned latent representation preserves neighboring similarity: spatially adjacent spots remain correlated in the low-dimensional embedding
- Downstream K-means (sklearn.cluster.KMeans) on Z recovers spatial domains consistent with manually annotated tissue structure (e.g. DLPFC layers)

## Limitations

- Current implementation has O(n^3) time and O(n^2) space complexity; authors aim to reduce both in future work
- Learning the lengthscale parameter may be challenging when data are sparse/noisy or the spatial structure is complicated; manual user tuning may be required
- The shared covariance assumption (Sigma_1=...=Sigma_d) is imposed for computational convenience; no closed-form solution exists when it is relaxed
- Operates on normalized data rather than raw counts, so dropout events and the mean-variance relationship of raw counts are not modeled, potentially reducing analytical power

## Evidence

- [readme] uses a kernel matrix to explicitly model the spatial correlation structure across tissue locations: [evidence span withheld — anchor status: external_doc]
