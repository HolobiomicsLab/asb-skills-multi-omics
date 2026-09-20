---
name: clustering-benchmark-evaluation
description: Use when a new dimension reduction/clustering method for paired spatial
  mRNA+ATAC data must be compared to competitors (chromVAR, cisTopic, LSA, PeakVI,
  SCALE, scMDC, Seurat, SpatialPCA) using label agreement and representation quality
  metrics.
when_to_use_negative:
- The analysis requires count-aware modeling of raw counts with dropout and mean–variance
  structure (e.g., PeakVI/TotalVI-style deep generative models) — SMOPCA uses normalized
  data, not raw expression measurements
- The dataset is very sparse/noisy or has very complicated spatial structure such
  that the kernel lengthscale parameter cannot be reliably learned without manual
  tuning
- 'The number of spots is very large: the current implementation has O(n^3) time and
  O(n^2) space complexity'
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SMOPCA
  role: 'Method under test: spatial multi-omics PCA producing the integrated low-dimensional
    representation and K-means cluster labels'
- name: SpatialPCA
  role: Competing spatially aware dimension reduction method whose latent representation
    and clustering are benchmarked
- name: scMDC
  role: Competing multimodal deep clustering method run on the same input features
  repo: https://github.com/xianglin226/scMDC
- name: Seurat V4
  role: Competing multimodal integration method benchmarked on the same dataset
  repo: https://github.com/satijalab/seurat
- name: scikit-learn KMeans
  role: Predicts final cluster labels from each method's low-dimensional vectors
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:b1eccf2be6e4e5700c190112a03c0c255395907db3f91d716a26e5a23b4b90f2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hemberg-lab/SC3
  repo_url: https://github.com/xianglin226/scMDC
  tools:
  - SMOPCA
  - Seurat V4
  - SpatialPCA
  - scMDC
  - scikit-learn
  - scikit-learn KMeans
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# clustering-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks a spatial multi-omics dimension reduction method (e.g., SMOPCA) against competing methods on the MISAR-seq E15.5 mouse embryonic brain dataset, scoring cluster labels with AMI, NMI, ARI and latent representations with the McFadden-adjusted pseudo R2.

## When to use

Use when a new dimension reduction/clustering method for paired spatial mRNA+ATAC data must be compared to competitors (chromVAR, cisTopic, LSA, PeakVI, SCALE, scMDC, Seurat, SpatialPCA) using label agreement and representation quality metrics.

## When NOT to use

- The analysis requires count-aware modeling of raw counts with dropout and mean–variance structure (e.g., PeakVI/TotalVI-style deep generative models) — SMOPCA uses normalized data, not raw expression measurements
- The dataset is very sparse/noisy or has very complicated spatial structure such that the kernel lengthscale parameter cannot be reliably learned without manual tuning
- The number of spots is very large: the current implementation has O(n^3) time and O(n^2) space complexity

## Inputs

- Preprocessed MISAR-seq E15.5 mouse embryonic brain spatial ATAC gene activity matrix (1949 spots retained of initial 2500)
- MISAR-seq mRNA count matrix restricted to SPARK spatially variable genes (FDR 0.05, Benjamini-Yekutieli)
- Spatial coordinates of the retained spots (kernel matrix input)
- Reference/manual cluster annotations for computing AMI, NMI, ARI

## Outputs

- Per-method AMI, NMI, and ARI scores written to benchmark_results.csv
- McFadden-adjusted pseudo R2 of each method's latent representation
- K-means cluster labels predicted from SMOPCA low-dimensional vectors

## How to apply

Load the preprocessed MISAR-seq E15.5 data (ATAC peaks collapsed to a gene activity matrix per the Satija lab protocol; 1949 of 2500 spots retained), then filter the mRNA modality to spatially variable genes selected by SPARK at FDR 0.05 with the Benjamini-Yekutieli procedure. Normalize each count matrix by library size, log-transform, and scale to zero mean and unit variance before fitting each method. Fit SMOPCA by eigen decomposition of the kernel matrix, iteratively estimating modality-specific (sigma_k, W_k) with gamma updated across modalities, and take the posterior mean of Z as the integrated low-dimensional representation; predict labels via sklearn.cluster.KMeans on these vectors. Run the eight competing methods on the same input features to obtain their latent representations and cluster labels, then compute AMI, NMI, ARI per method and the McFadden-adjusted pseudo R2 per latent representation, writing the per-method table to benchmark_results.csv.

## Related tools

- **SMOPCA** (Method under test: spatial multi-omics PCA producing the integrated low-dimensional representation and K-means cluster labels)
- **SpatialPCA** (Competing spatially aware dimension reduction method whose latent representation and clustering are benchmarked)
- **scMDC** (Competing multimodal deep clustering method run on the same input features) — https://github.com/xianglin226/scMDC
- **Seurat V4** (Competing multimodal integration method benchmarked on the same dataset) — https://github.com/satijalab/seurat
- **scikit-learn KMeans** (Predicts final cluster labels from each method's low-dimensional vectors)

## Examples

```
from sklearn.cluster import KMeans; cluster_labels = KMeans(n_clusters=n_domains).fit_predict(Z_posterior_mean)
```

## Evaluation signals

- SMOPCA consistently delivers the best results across all three clustering metrics (AMI, NMI, ARI) versus chromVAR, cisTopic, LSA, PeakVI, SCALE, scMDC, Seurat, and SpatialPCA
- McFadden-adjusted pseudo R2 of the SMOPCA latent representation exceeds that of every competing method's representation
- Preprocessing invariant: exactly 1949 of the initial 2500 MISAR-seq spots are retained after filtering
- mRNA features are restricted to SPARK-selected spatially variable genes at FDR 0.05 (Benjamini-Yekutieli) before any method is fit
- A per-method metric table is produced and written to benchmark_results.csv

## Limitations

- Current implementation has O(n^3) time and O(n^2) space complexity; authors aim to reduce both in future work
- SMOPCA does not model dropout events or the mean–variance relationship of raw counts and uses normalized rather than raw data, which may reduce analytical power
- Learning the kernel lengthscale parameter may be challenging when data are sparse/noisy or spatial structure is complicated, possibly requiring manual tuning
- The shared covariance assumption (Sigma_1=...=Sigma_d) is imposed for computational convenience; relaxing it requires Stiefel-manifold constrained optimization deferred to future work

## Evidence

- [readme] SpatialPCA role as competitor: [evidence span withheld — anchor status: external_doc]
