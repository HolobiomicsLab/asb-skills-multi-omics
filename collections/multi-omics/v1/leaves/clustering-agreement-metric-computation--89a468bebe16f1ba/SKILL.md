---
name: clustering-agreement-metric-computation
description: Use when benchmarking a clustering or dimension-reduction pipeline (e.g.,
  SMOPCA vs SpatialPCA, scMDC, PCA+K-means) against known ground-truth cell-type labels
  and you need AMI, NMI, and ARI per replicate and dataset.
when_to_use_negative:
- When no ground-truth cell-type labels exist — AMI/NMI/ARI require reference labels
  and cannot be computed on purely unsupervised spatial domain detections.
- When the goal is spatial domain detection on real spatial transcriptomics data rather
  than single-cell data with simulated 2D grid coordinates — other evaluation layers
  of the SMOPCA study apply.
- When comparing methods that operate on raw counts vs normalized data without harmonizing
  preprocessing (library-size normalization, log transformation, scaling), since metric
  differences may reflect preprocessing rather than clustering quality.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SMOPCA
  role: Produces the integrated latent representation (posterior mean of Z) whose
    K-means cluster labels are scored with AMI/NMI/ARI
- name: scikit-learn
  role: Provides sklearn.cluster.KMeans for label prediction on low-dimensional vectors;
    sklearn metrics back AMI/NMI/ARI computation
- name: SpatialPCA
  role: Competing baseline applied to concatenated modalities on simulated coordinates;
    its labels are scored with the same metrics
  repo: https://github.com/shangll123/SpatialPCA
- name: scMDC
  role: Competing multimodal deep clustering method whose labels are scored with AMI/NMI/ARI
    on the same simulated datasets
  repo: https://github.com/xianglin226/scMDC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:c2f07d6550137b0ef60ac92a74742f915941ae37335200971a5874d870762fba
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
yes_rate: 1.0
n_claims: 5
---

# clustering-agreement-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies agreement between predicted clustering labels (e.g., from SMOPCA latent vectors via K-means) and ground-truth cell-type labels using AMI, NMI, and ARI, aggregated over repeated simulation replicates. This is the standard evaluation layer for benchmarking dimension-reduction and clustering methods on multi-omics datasets.

## When to use

Use when benchmarking a clustering or dimension-reduction pipeline (e.g., SMOPCA vs SpatialPCA, scMDC, PCA+K-means) against known ground-truth cell-type labels and you need AMI, NMI, and ARI per replicate and dataset.

## When NOT to use

- When no ground-truth cell-type labels exist — AMI/NMI/ARI require reference labels and cannot be computed on purely unsupervised spatial domain detections.
- When the goal is spatial domain detection on real spatial transcriptomics data rather than single-cell data with simulated 2D grid coordinates — other evaluation layers of the SMOPCA study apply.
- When comparing methods that operate on raw counts vs normalized data without harmonizing preprocessing (library-size normalization, log transformation, scaling), since metric differences may reflect preprocessing rather than clustering quality.

## Inputs

- Predicted cluster label vector per cell/spot (e.g., sklearn.cluster.KMeans output on SMOPCA latent vectors)
- Ground-truth cell-type label vector per cell/spot
- Per-replicate and per-dataset identifiers (7 simulated datasets × 10 replicates)
- Preprocessed count matrices (library-size normalized, log-transformed, scaled to zero mean and unit variance)

## Outputs

- AMI, NMI, and ARI values for every method, dataset, and replicate
- Per-dataset metric distributions aggregated across replicates (simulated_spatial_clustering_metrics.csv)

## How to apply

After each method produces cluster assignments — for SMOPCA, sklearn.cluster.KMeans is applied to the posterior mean of the latent Z used as the integrated low-dimensional representation — compare predicted labels with ground-truth cell-type labels by computing Adjusted Mutual Information (AMI), Normalized Mutual Information (NMI), and Adjusted Rand Index (ARI). Because the SMOPCA simulation study repeats the procedure 10 times per dataset, compute the three metrics for every replicate and dataset combination to yield per-dataset metric distributions (saved as simulated_spatial_clustering_metrics.csv). Use the distributions (not a single run) to compare methods, e.g., SMOPCA vs SpatialPCA applied to concatenated modalities vs scMDC vs PCA + K-means baselines across the five CITE-seq (PBMC, SLN111D1, SLN111D2, SLN208D1, SLN208D2) and two SMAGE-seq (PBMC3K, PBMC10K) datasets with simulated 2D grid coordinates. Ground-truth labels come from the published cell-type annotations accompanying each dataset.

## Related tools

- **SMOPCA** (Produces the integrated latent representation (posterior mean of Z) whose K-means cluster labels are scored with AMI/NMI/ARI)
- **scikit-learn** (Provides sklearn.cluster.KMeans for label prediction on low-dimensional vectors; sklearn metrics back AMI/NMI/ARI computation)
- **SpatialPCA** (Competing baseline applied to concatenated modalities on simulated coordinates; its labels are scored with the same metrics) — https://github.com/shangll123/SpatialPCA
- **scMDC** (Competing multimodal deep clustering method whose labels are scored with AMI/NMI/ARI on the same simulated datasets) — https://github.com/xianglin226/scMDC

## Examples

```
from sklearn.cluster import KMeans; from sklearn.metrics import adjusted_mutual_info_score, normalized_mutual_info_score, adjusted_rand_score; labels = KMeans(n_clusters=k, random_state=r).fit_predict(Z_posterior_mean); ami = adjusted_mutual_info_score(y_true, labels)
```

## Evaluation signals

- All three metrics (AMI, NMI, ARI) are computed for every method × dataset × replicate combination — no missing cells in the metric table.
- SMOPCA should outperform SpatialPCA (on concatenated modalities) across all three clustering metrics on the simulated single-cell multi-omics datasets.
- Per-dataset metric distributions over 10 replicates should show lower variance than single-run evaluations; large spread indicates unstable clustering.
- Metric values fall in valid ranges: AMI and NMI in [0, 1]; ARI can be negative for random assignments and near 0 for independent labelings.
- Metrics computed against the published ground-truth cell-type annotations, not against a competing method's labels.

## Limitations

- Single-cell multi-omics data were simulated spatial coordinates by allocating cells row by row and column by column onto a 2D grid, so the benchmark reflects artificial spatial structure rather than real tissue geometry.
- Results depend on preprocessing choices: library-size normalization, log transformation, and scaling to zero mean and unit variance; methods using raw counts (e.g., count-aware deep generative alternatives) may behave differently.
- AMI/NMI/ARI capture label agreement only and do not reward spatial coherence of domains or preserve of neighboring similarity.

## Evidence
