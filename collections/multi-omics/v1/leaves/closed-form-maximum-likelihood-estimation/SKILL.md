---
name: closed-form-maximum-likelihood-estimation
description: Use when implementing SMOPCA or a GPPCA-style spatial factor model where
  closed-form maximum marginal likelihood updates for per-modality parameters (sigma_k,
  W_k) are available and iterative sweeps across modalities with gamma updates are
  required.
when_to_use_negative:
- 'When the shared-covariance assumption (Sigma_1=...=Sigma_d) must be relaxed: no
  closed-form solution exists and Stiefel-manifold-constrained numerical optimization
  would be needed'
- 'When raw-count mean-variance relationships or dropout events must be modeled: SMOPCA-style
  MLE operates on normalized data, unlike count-aware deep generative alternatives
  such as totalVI or spaVAE'
- When spatial structure is very complicated or data are sparse and noisy, making
  automatic lengthscale learning unreliable (manual tuning may be needed)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3934
- http://edamontology.org/topic_3170
tools:
- name: SpatialPCA
  role: State-of-the-art spatially aware dimension reduction method that SMOPCA builds
    upon via generalized probabilistic principal component analysis (GPPCA); reference
    implementation for kernel-based spatial covariance modeling
  repo: https://github.com/shangll123/SpatialPCA
- name: SRTsim
  role: Used to generate simulated 3-omics datasets from DLPFC reference preserving
    spatial patterns, providing test inputs for validating the closed-form MLE pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:12356399fdd33f57fcfdc90500ba92938f5d00885446b1936dc7ab4b8483564f
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
  - SRTsim
  - SpatialPCA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3934
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# closed-form-maximum-likelihood-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A parameter-estimation technique in which modality-specific model parameters (noise variance sigma_k and loading matrix W_k) of the SMOPCA multimodal probabilistic PCA model are obtained analytically by maximizing the marginal likelihood, rather than via gradient-based numerical optimization, with the latent precision gamma updated between modalities.

## When to use

Use when implementing SMOPCA or a GPPCA-style spatial factor model where closed-form maximum marginal likelihood updates for per-modality parameters (sigma_k, W_k) are available and iterative sweeps across modalities with gamma updates are required.

## When NOT to use

- When the shared-covariance assumption (Sigma_1=...=Sigma_d) must be relaxed: no closed-form solution exists and Stiefel-manifold-constrained numerical optimization would be needed
- When raw-count mean-variance relationships or dropout events must be modeled: SMOPCA-style MLE operates on normalized data, unlike count-aware deep generative alternatives such as totalVI or spaVAE
- When spatial structure is very complicated or data are sparse and noisy, making automatic lengthscale learning unreliable (manual tuning may be needed)

## Inputs

- Normalized expression matrices for two or more modalities (e.g., RNA and protein from spatial-CITE-seq), scaled to zero mean and unit variance
- Spatial coordinates for each spot (used to build the shared Matern kernel covariance)
- Kernel matrix Sigma with eigen decomposition for scalable computation

## Outputs

- Modality-specific loading matrices W_k and noise variances sigma_k
- Updated latent precision gamma
- Posterior mean of latent factors Z (n_spots x n_factors integrated representation)
- Learned kernel lengthscale from marginal likelihood maximization

## How to apply

Evaluate the shared spatial covariance kernel (e.g., Matern nu=3/2 on spot coordinates, with eigen decomposition of the kernel matrix for scalability), then iterate over modalities: for each modality k, update sigma_k and W_k using the closed-form maximum marginal likelihood expressions while holding other parameters fixed, and update gamma (the latent-factor precision) between modalities. Repeat sweeps until convergence, then learn the kernel lengthscale by maximizing the model marginal likelihood and compute the posterior mean of Z as the integrated low-dimensional representation. The closed form exists only under the shared-covariance assumption (Sigma_1 = ... = Sigma_d); relaxing it requires orthogonal-constrained numerical optimization on the Stiefel manifold. Input data should be library-size-normalized, log-transformed, and scaled to zero mean and unit variance.

## Related tools

- **SpatialPCA** (State-of-the-art spatially aware dimension reduction method that SMOPCA builds upon via generalized probabilistic principal component analysis (GPPCA); reference implementation for kernel-based spatial covariance modeling) — https://github.com/shangll123/SpatialPCA
- **SRTsim** (Used to generate simulated 3-omics datasets from DLPFC reference preserving spatial patterns, providing test inputs for validating the closed-form MLE pipeline)

## Examples

```
Sigma = kern(x, x, nu=1.5, lengthscale=l); for k in modalities: sigma_k, W_k = mle_update(Sigma, X_k, gamma); gamma = update_gamma(W_k, sigma_k); Z = posterior_mean(Sigma, Ws, sigmas)
```

## Evaluation signals

- Z has shape (n_spots, n_factors) with all finite values
- Learned lengthscale is positive and maximizes (does not decrease) the model marginal likelihood across iterations
- Iterative sigma_k/W_k updates converge (marginal likelihood plateaus across modality sweeps with gamma updated between modalities)
- K-means clustering (sklearn.cluster.KMeans) on the resulting low-dimensional vectors yields spatial domains consistent with known spatial correlation structure

## Limitations

- Current implementation has O(n^3) time and O(n^2) space complexity due to kernel-matrix operations; authors aim to reduce both in future work
- The closed-form solution depends on the shared covariance assumption (Sigma_1=...=Sigma_d); relaxing it requires orthogonal-constrained (Stiefel manifold) numerical optimization, deferred to future work
- Learning the lengthscale parameter may be challenging when data are very sparse and noisy or spatial structure is very complicated, potentially requiring manual user tuning
- Method does not model dropout events or the mean-variance relationship of raw counts and uses normalized data, which may reduce analytical power

## Evidence

- [methods] Iterative MLE of modality-specific parameters: "The modality-specific parameters"
- [discussion] Shared covariance assumption: "However, unlike the case where the covariance is shared, no closed-form solution for"
- [discussion] Lengthscale learning difficulty: "when the data are very sparse and noisy, or the spatial structure is very complicated, it may be challenging to learn the appropriate lengthscale parameter"
- [discussion] Complexity limitation: "In our future work, we aim to further optimize the multi-omics model to reduce both time and space complexity."
