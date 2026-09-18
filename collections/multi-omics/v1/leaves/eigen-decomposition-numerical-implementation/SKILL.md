---
name: eigen-decomposition-numerical-implementation
description: Use when implementing a spatially aware factor-analysis/PCA model whose
  Gaussian-process covariance over spot coordinates would otherwise require O(n^3)
  operations on the dense kernel matrix, and a low-rank spectral representation is
  needed.
when_to_use_negative:
- Data are single-cell multi-omics with genuine spatial coordinates unavailable and
  no pseudo-spatial coordinates (e.g. UMAP-derived) have been generated, so no spatial
  kernel can be built
- The analysis goal is clustering without spatial dependency modeling (e.g. plain
  scRNA-seq clustering with BREM-SC or scMDC), where the GP covariance structure adds
  no value
- 'A per-modality covariance is required: SMOPCA imposes the shared-covariance assumption
  (Sigma_1=...=Sigma_d) for computational convenience, and no closed-form solution
  exists when it is relaxed'
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
tools:
- name: SpatialPCA
  role: Predecessor method that introduced using a kernel matrix to explicitly model
    spatial correlation across tissue locations; SMOPCA's eigen decomposition of the
    spatial kernel follows this design
  repo: https://github.com/shangll123/SpatialPCA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:47f90f614d54449982d81a2f068d845115370e011710b6eb9a7b998159617af6
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# eigen-decomposition-numerical-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Numerical technique used by SMOPCA (and its predecessor SpatialPCA) to make the shared spatial Gaussian-process covariance computationally tractable: the Matern kernel matrix built on spot coordinates is eigendecomposed so the dense covariance enters the model only through a small number of eigenvalues/eigenvectors.

## When to use

Use when implementing a spatially aware factor-analysis/PCA model whose Gaussian-process covariance over spot coordinates would otherwise require O(n^3) operations on the dense kernel matrix, and a low-rank spectral representation is needed.

## When NOT to use

- Data are single-cell multi-omics with genuine spatial coordinates unavailable and no pseudo-spatial coordinates (e.g. UMAP-derived) have been generated, so no spatial kernel can be built
- The analysis goal is clustering without spatial dependency modeling (e.g. plain scRNA-seq clustering with BREM-SC or scMDC), where the GP covariance structure adds no value
- A per-modality covariance is required: SMOPCA imposes the shared-covariance assumption (Sigma_1=...=Sigma_d) for computational convenience, and no closed-form solution exists when it is relaxed

## Inputs

- Spatial coordinates matrix for each spot/cell (n_spots x 2 or 3)
- Matern nu=3/2 kernel matrix Sigma evaluated on the spot coordinates
- Choice of number of eigenpairs to retain for the low-rank approximation

## Outputs

- Eigenvalues and eigenvectors of the kernel matrix Sigma
- Low-rank spectral representation of the shared spatial covariance used in the closed-form MLE updates
- Posterior mean latent factor matrix Z (n_spots, n_factors) derived using the decomposed covariance

## How to apply

First build the shared spatial covariance by evaluating a Matern nu=3/2 kernel on the spot coordinates, then perform eigen decomposition on the resulting kernel matrix for scalable computation. In SMOPCA the spectral form feeds into closed-form maximum marginal likelihood updates of the modality-specific parameters (sigma_k, W_k), iterating across modalities with the latent gamma updated between modalities, while the kernel lengthscale is learned by maximizing the model marginal likelihood. The eigen decomposition is what allows the posterior mean of the latent factors Z to be computed efficiently as the integrated low-dimensional representation. Validate the result by checking that the retained eigen representation reproduces the covariance structure (neighboring spot similarity) and that downstream Z has shape (n_spots, n_factors) with finite values.

## Related tools

- **SpatialPCA** (Predecessor method that introduced using a kernel matrix to explicitly model spatial correlation across tissue locations; SMOPCA's eigen decomposition of the spatial kernel follows this design) — https://github.com/shangll123/SpatialPCA

## Examples

```
import numpy as np; K = matern32_kernel(coords, lengthscale=l); eigvals, eigvecs = np.linalg.eigh(K); Sigma = (eigvecs * eigvals) @ eigvecs.T
```

## Evaluation signals

- The eigendecomposed covariance must reproduce the spatial correlation structure: learned latent representations should preserve neighboring similarity of the original spots
- Latent factor matrix Z output must have shape (n_spots, n_factors) and contain only finite values
- Marginal likelihood should improve monotonically as closed-form updates of (sigma_k, W_k) iterate across modalities with gamma updated between modalities
- The learned lengthscale from maximizing the marginal likelihood should yield a Matern kernel whose correlations decay reasonably with inter-spot distance (manual tuning indicates failure when it does not)

## Limitations

- Current implementation has O(n^3) time and O(n^2) space complexity; the authors aim to reduce both in future work
- Learning the lengthscale parameter may be challenging when data are sparse/noisy or spatial structure is complicated; manual user tuning may be required
- The shared covariance assumption (Sigma_1=...=Sigma_d) is imposed for computational convenience; relaxing it requires orthogonal-constrained (Stiefel manifold) numerical optimization with no closed-form solution

## Evidence

- [intro] SpatialPCA kernel-based spatial correlation modeling: [evidence span withheld — anchor status: external_doc]
