---
name: marginal-likelihood-hyperparameter-optimization
description: Use when fitting a SMOPCA-style Gaussian spatial latent-factor model
  to multi-omics matrices with spot coordinates, and the Matérn kernel lengthscale
  gamma must be learned by maximizing the closed-form marginal likelihood rather than
  set by hand.
when_to_use_negative:
- The data are very sparse and noisy or the spatial structure is very complicated
  — the authors warn that learning the appropriate lengthscale parameter may then
  be challenging, and manual user tuning may be required.
- You need to relax the shared spatial covariance assumption (Sigma_1=...=Sigma_d)
  across modalities — no closed-form marginal-likelihood solution exists, and the
  required orthogonal-constrained (Stiefel manifold) optimization was deferred to
  future work.
- The analysis requires raw-count modeling of dropout events and the mean–variance
  relationship — SMOPCA uses normalized data rather than raw expression measurements,
  so count-aware generative methods are preferable.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3914
tools:
- name: SpatialPCA
  role: State-of-the-art GPPCA-based spatial PCA that uses a kernel matrix to explicitly
    model spatial correlation; closest single-modality analogue to SMOPCA's spatially-aware
    marginal-likelihood framework, used as a comparison baseline.
- name: MEFISTO
  role: MOFA2-based multi-omics latent factor model that natively handles three or
    more modalities; alternative framework contrasted with SMOPCA's closed-form marginal-likelihood
    estimation (less competitive performance and significantly longer training time).
  repo: https://biofam.github.io/MOFA2/MEFISTO
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:fe3923944682328d2f42c67d094931b7661f438c5f94ccfb2388b43915faa177
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hemberg-lab/SC3
  tools:
  - MEFISTO
  - SpatialPCA
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3914
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# marginal-likelihood-hyperparameter-optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

In SMOPCA, all model parameters — the Matérn nu=3/2 kernel lengthscale gamma of the shared spatial covariance, and the modality-specific noise variances sigma_k and loadings W_k — are estimated by maximum marginal likelihood. Because the latent-factor model is fully Gaussian, the marginal likelihood is closed-form, so the spatial scale of the kernel is learned automatically from data instead of being hand-tuned.

## When to use

Use when fitting a SMOPCA-style Gaussian spatial latent-factor model to multi-omics matrices with spot coordinates, and the Matérn kernel lengthscale gamma must be learned by maximizing the closed-form marginal likelihood rather than set by hand.

## When NOT to use

- The data are very sparse and noisy or the spatial structure is very complicated — the authors warn that learning the appropriate lengthscale parameter may then be challenging, and manual user tuning may be required.
- You need to relax the shared spatial covariance assumption (Sigma_1=...=Sigma_d) across modalities — no closed-form marginal-likelihood solution exists, and the required orthogonal-constrained (Stiefel manifold) optimization was deferred to future work.
- The analysis requires raw-count modeling of dropout events and the mean–variance relationship — SMOPCA uses normalized data rather than raw expression measurements, so count-aware generative methods are preferable.

## Inputs

- Normalized multi-modal expression matrices (library-size normalized, log-transformed, scaled to zero mean and unit variance), one per modality (e.g., CITE-seq genes and proteins, SMAGE-seq genes)
- 2D spatial coordinates for each spot, or pseudo-spatial coordinates from UMAP (top 50 PCs) / row-by-row 2D grid assignment for single-cell multi-omics data
- Number of latent factors and an initial value for the Matern nu=3/2 kernel lengthscale gamma

## Outputs

- Learned kernel lengthscale gamma obtained by maximizing the model marginal likelihood
- Closed-form maximum-likelihood estimates of the modality-specific noise variances sigma_k and loading matrices W_k
- Posterior mean of latent factors Z (n_spots x n_factors) — the joint low-dimensional representation used for K-means spatial domain detection

## How to apply

Preprocess each modality by library-size normalization, log transformation, and scaling to zero mean and unit variance, then build the shared spatial covariance by evaluating a Matern nu=3/2 kernel on the 2D spot coordinates and eigen-decomposing the kernel matrix Sigma for scalable computation. Because the model is fully Gaussian, integrating out the latent factors yields a closed-form marginal likelihood, from which closed-form maximum-likelihood updates of the modality-specific parameters sigma_k and W_k are derived; iterate across modalities with gamma (the kernel lengthscale) updated between modality updates so it is learned automatically by maximizing the model marginal likelihood. When only single-cell multi-omics data are available, supply pseudo-spatial coordinates generated from UMAP on the top 50 PCs, or assign cells to a 2D grid row by row. After convergence, compute the posterior mean of Z as the integrated low-dimensional representation and run sklearn.cluster.KMeans on it to predict spatial domain labels, validating that Z has shape (n_spots, n_factors) with finite values.

## Related tools

- **SpatialPCA** (State-of-the-art GPPCA-based spatial PCA that uses a kernel matrix to explicitly model spatial correlation; closest single-modality analogue to SMOPCA's spatially-aware marginal-likelihood framework, used as a comparison baseline.)
- **MEFISTO** (MOFA2-based multi-omics latent factor model that natively handles three or more modalities; alternative framework contrasted with SMOPCA's closed-form marginal-likelihood estimation (less competitive performance and significantly longer training time).) — https://biofam.github.io/MOFA2/MEFISTO

## Examples

```
import numpy as np; w, V = np.linalg.eigh(Sigma)  # Sigma: Matern nu=3/2 kernel matrix on spot coords; lengthscale gamma tuned by maximizing marginal likelihood
```

## Evaluation signals

- The marginal likelihood should improve as the closed-form sigma_k and W_k updates are alternated across modalities with gamma updated between modalities, until the learned lengthscale converges.
- The learned gamma must yield a shared kernel covariance whose eigen decomposition supports scalable computation while the posterior Z preserves the neighboring similarity of the original spots.
- Output check: the joint latent factor matrix Z saved to latent_factors.csv must have shape (n_spots, n_factors) with all-finite values, alongside the learned lengthscale and the modality-specific loadings W_k.
- Downstream sklearn.cluster.KMeans on the low-dimensional vectors should recover spatially coherent domains, e.g. the known layers across the 12 DLPFC sections (151507–151676).
- Benchmark expectation: with the learned lengthscale, SMOPCA should outperform existing single-modal and multimodal dimension reduction and clustering methods across both single-cell and spatial multi-omics datasets.

## Limitations

- Learning the lengthscale parameter may be challenging when the data are very sparse and noisy or the spatial structure is very complicated; manual user tuning may be required.
- The shared covariance assumption (Sigma_1=...=Sigma_d) is imposed for computational convenience; no closed-form solution exists when it is relaxed, and relaxing it would require orthogonal-constrained (Stiefel manifold) numerical optimization deferred to future work.
- Current implementation has O(n^3) time and O(n^2) space complexity; the authors aim to reduce both in future work.
- Inputs are normalized data rather than raw expression measurements, and dropout events and the inherent mean–variance relationship of raw counts are not modeled, potentially reducing analytical power.

## Evidence

- [readme] Kernel matrix explicitly models spatial correlation across tissue locations (closest analogue, SpatialPCA): [evidence span withheld — anchor status: external_doc]
