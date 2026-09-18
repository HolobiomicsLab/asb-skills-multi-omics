---
name: latent-representation-pseudo-r2-evaluation
description: Use when comparing latent embeddings from competing multi-omics dimension
  reduction methods on the same spots/cells and you need clustering agreement (AMI/NMI/ARI)
  plus latent quality (McFadden pseudo R2) in one table.
when_to_use_negative:
- Input is raw unnormalized counts where dropout and mean-variance modeling matter
  — SMOPCA uses normalized data rather than raw expression measurements, so count-aware
  deep generative alternatives may be preferable
- Data with no spatial coordinates or spot-level layout, since the kernel matrix and
  spatial correlation modeling central to this benchmark assume spatially resolved
  observations
- Datasets too large for the current implementation given its O(n^3) time and O(n^2)
  space complexity
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn
  role: Provides sklearn.cluster.KMeans for predicting cluster labels on low-dimensional
    vectors and the metrics for AMI, NMI, ARI
- name: SpatialPCA
  role: State-of-the-art spatial dimension reduction baseline whose latent representation
    and cluster labels are scored in the same metric table
  repo: https://github.com/shangll123/SpatialPCA
- name: scMDC
  role: Multimodal deep clustering baseline run on the same input features to obtain
    latent representations and cluster labels
  repo: https://github.com/xianglin226/scMDC
- name: chromVAR
  role: ATAC motif-variability baseline contributing cluster labels and embeddings
    to the benchmark comparison
  repo: https://github.com/GreenleafLab/chromVAR
- name: cisTopic
  role: Epigenomic topic-modeling baseline for latent representation and clustering
    comparison
  repo: https://github.com/aertslab/pycisTopic
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:8b7d21be14c19de1ad777464db11afa98d93ac4271df200e198067e852727795
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
  - chromVAR
  - cisTopic
  - scMDC
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# latent-representation-pseudo-r2-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark dimension-reduction/clustering methods on spatial multi-omics data by scoring their cluster labels (AMI, NMI, ARI against reference labels) and the McFadden-adjusted pseudo R2 of their latent representations, as done for SMOPCA versus chromVAR, cisTopic, LSA, PeakVI, SCALE, scMDC, Seurat, and SpatialPCA on MISAR-seq E15.5 mouse brain.

## When to use

Use when comparing latent embeddings from competing multi-omics dimension reduction methods on the same spots/cells and you need clustering agreement (AMI/NMI/ARI) plus latent quality (McFadden pseudo R2) in one table.

## When NOT to use

- Input is raw unnormalized counts where dropout and mean-variance modeling matter — SMOPCA uses normalized data rather than raw expression measurements, so count-aware deep generative alternatives may be preferable
- Data with no spatial coordinates or spot-level layout, since the kernel matrix and spatial correlation modeling central to this benchmark assume spatially resolved observations
- Datasets too large for the current implementation given its O(n^3) time and O(n^2) space complexity

## Inputs

- Preprocessed MISAR-seq E15.5 mouse embryonic brain data (gene activity matrix from ATAC peaks + mRNA matrix; 1949 of 2500 spots retained)
- Latent low-dimensional representations (posterior mean of Z) from each compared method
- Predicted cluster labels from sklearn.cluster.KMeans on each method's embeddings
- Reference/ground-truth spatial domain labels for AMI/NMI/ARI computation

## Outputs

- Per-method table of AMI, NMI, ARI, and McFadden-adjusted pseudo R2 written to benchmark_results.csv
- Ranking showing whether SMOPCA consistently bests chromVAR, cisTopic, LSA, PeakVI, SCALE, scMDC, Seurat, and SpatialPCA across all three clustering metrics

## How to apply

First place all methods on identical inputs: e.g. the MISAR-seq E15.5 mouse brain data where the ATAC peak matrix is collapsed to a gene activity matrix (Satija lab protocol), 1949 of 2500 spots retained, mRNA restricted to SPARK spatially variable genes at FDR 0.05 under Benjamini-Yekutieli, and each count matrix library-size normalized, log transformed, and scaled to zero mean/unit variance. Obtain each method's latent representation (for SMOPCA: eigen decomposition of the kernel matrix, iterative MLE of sigma_k and W_k with gamma updated across modalities, then the posterior mean of Z). Predict cluster labels with K-means (sklearn.cluster.KMeans) on the low-dimensional vectors, and run competing methods chromVAR, cisTopic, LSA, PeakVI, SCALE, scMDC, Seurat, and SpatialPCA on the same features to get their embeddings and labels. Compute AMI, NMI, and ARI for each method's clustering versus the reference labels, and the McFadden-adjusted pseudo R2 of each latent representation, then write the per-method metric table to benchmark_results.csv. Judge success when SMOPCA (or the method under test) consistently ranks best across all three clustering metrics and its embedding yields the highest pseudo R2.

## Related tools

- **scikit-learn** (Provides sklearn.cluster.KMeans for predicting cluster labels on low-dimensional vectors and the metrics for AMI, NMI, ARI)
- **SpatialPCA** (State-of-the-art spatial dimension reduction baseline whose latent representation and cluster labels are scored in the same metric table) — https://github.com/shangll123/SpatialPCA
- **scMDC** (Multimodal deep clustering baseline run on the same input features to obtain latent representations and cluster labels) — https://github.com/xianglin226/scMDC
- **chromVAR** (ATAC motif-variability baseline contributing cluster labels and embeddings to the benchmark comparison) — https://github.com/GreenleafLab/chromVAR
- **cisTopic** (Epigenomic topic-modeling baseline for latent representation and clustering comparison) — https://github.com/aertslab/pycisTopic

## Examples

```
from sklearn.cluster import KMeans; from sklearn.metrics import adjusted_mutual_info_score, normalized_mutual_info_score, adjusted_rand_score; labels = KMeans(n_clusters=7).fit_predict(Z_posterior_mean)
```

## Evaluation signals

- benchmark_results.csv contains one row per method with finite AMI, NMI, ARI, and McFadden pseudo R2 values computed on the identical 1949-spot input set
- Same method ordering is stable across AMI, NMI, and ARI (SMOPCA consistently best across all three clustering metrics per the paper's finding)
- Clustering labels are regenerated via KMeans on each method's low-dimensional vectors rather than taken from heterogeneous native cluster outputs
- Preprocessing invariants hold: 1949 of 2500 spots retained, mRNA features are SPARK SV genes at FDR 0.05 (BY procedure), matrices scaled to zero mean and unit variance
- McFadden pseudo R2 is the McFadden-adjusted variant computed on the latent representation, not on raw expression profiles

## Limitations

- SMOPCA's current implementation has O(n^3) time and O(n^2) space complexity, which the authors aim to reduce in future work
- The method uses normalized data rather than raw counts and does not model dropout events or the mean-variance relationship, potentially reducing analytical power
- Learning the lengthscale parameter may be challenging when data are sparse/noisy or spatial structure is complicated, possibly requiring manual user tuning
- A shared covariance assumption (Σ1=...=Σd) is imposed for computational convenience; relaxing it has no closed-form solution and would need Stiefel-manifold optimization deferred to future work

## Evidence

- [readme] SpatialPCA baseline description: [evidence span withheld — anchor status: external_doc]
