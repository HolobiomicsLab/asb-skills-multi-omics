---
name: robust-rank-aggregation-statistics
description: Use when comparing several integration/clustering methods across multiple
  datasets and metrics and you need a single, statistically defensible overall ranking
  rather than per-metric cherry-picking.
when_to_use_negative:
- You have only a single method and dataset with no competitors to rank — direct metric
  reporting suffices
- Methods are evaluated on non-comparable task definitions or different data splits,
  making cross-method rank aggregation meaningless
- You need calibration of absolute clustering quality rather than relative ordering
  of methods
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SCANPY
  role: Clustering of every integrated embedding with the Leiden algorithm, sweeping
    resolution 0.1-1.0 to generate the replicate labels that feed the ranking metrics
- name: SSGATE
  role: Method under test; its dual-path graph attention auto-encoder embeddings are
    ranked against baselines with default hyperparameters (300 epochs, out_dims 30,
    lr 0.001)
  repo: https://github.com/Linliu-Bioinf/SSGATE
- name: MOFA+
  role: Baseline method whose embeddings contribute rank lists to the average ranking
    and RRA computation
- name: Seurat v5
  role: Network-based baseline (weighted nearest neighbor graph) contributing rank
    lists to the aggregation
- name: TotalVI
  role: Probabilistic VAE baseline whose clustering metrics are ranked alongside the
    other methods
- name: ScMM
  role: Mixture-of-experts multimodal VAE baseline contributing rank lists to the
    aggregation
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:2861943cd811e6228068bf11e0ff83a31df761fba43ef3209dd71a7081402b25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Linliu-Bioinf/SSGATE
  tools:
  - MOFA
  - MOFA+
  - SCANPY
  - SSGATE
  - ScMM
  - Seurat v5
  - TotalVI
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# robust-rank-aggregation-statistics

## Summary

Benchmark comparison protocol that ranks competing methods on each evaluation metric per dataset (Purity, Homogeneity Score, ARI, NMI), computes cross-metric average rankings, and derives Robust Rank Aggregation (RRA) p-values to declare an overall winner (e.g., SSGATE vs. MOFA+, Seurat v5, totalVI, scMM).

## When to use

Use when comparing several integration/clustering methods across multiple datasets and metrics and you need a single, statistically defensible overall ranking rather than per-metric cherry-picking.

## When NOT to use

- You have only a single method and dataset with no competitors to rank — direct metric reporting suffices
- Methods are evaluated on non-comparable task definitions or different data splits, making cross-method rank aggregation meaningless
- You need calibration of absolute clustering quality rather than relative ordering of methods

## Inputs

- Per-method, per-dataset, per-resolution clustering labels from Leiden (resolutions 0.1-1.0, 10 replicates per method per dataset)
- Ground-truth cell-type labels for BMNC and SLN111_D1 paired transcriptome-proteome datasets
- Integrated embeddings from SSGATE, MOFA+, Seurat v5, totalVI, and scMM run on identically preprocessed h5ad inputs

## Outputs

- benchmark_metrics.csv containing Purity, HS, ARI, NMI for every method/dataset/resolution combination
- Average rankings across all metrics on both datasets per method
- Robust Rank Aggregation (RRA) p-values per method
- noise_robustness_results.csv with metrics averaged over three Gaussian-noise levels

## How to apply

For each dataset (e.g., BMNC and SLN111_D1 paired transcriptome-proteome h5ad data), cluster every method's integrated embedding with the Leiden algorithm in SCANPY, sweeping the resolution parameter over 0.1, 0.2, ..., 1.0 to yield 10 independent replicates per method per dataset. Compute Purity (P), Homogeneity Score (HS), Adjusted Rand Index (ARI), and Normalized Mutual Information (NMI) against ground-truth labels for every method/dataset/resolution combination. Rank the five methods on each metric per dataset, then average the ranks across all metrics on both datasets to obtain the comprehensive average ranking (expected: SSGATE 1.5 on BMNC, 2.25 tied with totalVI on SLN111_D1). Finally calculate Robust Rank Aggregation (RRA) p-values over the rank lists, with the smallest RRA p-value among methods (SSGATE in the paper) identifying the most consistently top-ranked method. Save the full metrics table and the average rankings plus RRA p-values to benchmark_metrics.csv; for the noise-robustness variant, average metrics across three Gaussian-noise levels first (expected comprehensive rankings: scMM 4.88, totalVI 3.63, Seurat v5 2.38, MOFA+ 2.25, SSGATE 1.88).

## Related tools

- **SCANPY** (Clustering of every integrated embedding with the Leiden algorithm, sweeping resolution 0.1-1.0 to generate the replicate labels that feed the ranking metrics)
- **SSGATE** (Method under test; its dual-path graph attention auto-encoder embeddings are ranked against baselines with default hyperparameters (300 epochs, out_dims 30, lr 0.001)) — https://github.com/Linliu-Bioinf/SSGATE
- **MOFA+** (Baseline method whose embeddings contribute rank lists to the average ranking and RRA computation)
- **Seurat v5** (Network-based baseline (weighted nearest neighbor graph) contributing rank lists to the aggregation)
- **TotalVI** (Probabilistic VAE baseline whose clustering metrics are ranked alongside the other methods)
- **ScMM** (Mixture-of-experts multimodal VAE baseline contributing rank lists to the aggregation)

## Examples

```
sc.tl.leiden(adata, resolution=r, key_added=f'leiden_{r}')  # for r in [0.1..1.0]; then compute purity, homogeneity_score, adjusted_rand_score, normalized_mutual_info_score vs. ground truth, rank methods per metric, and aggregate via RRA
```

## Evaluation signals

- SSGATE's average ranking on BMNC equals 1.5 (first in ARI and NMI, second in Purity and HS)
- On SLN111_D1 SSGATE attains an average ranking of 2.25, tied with totalVI
- SSGATE has the smallest RRA p-value among the five methods
- Under Gaussian noise the comprehensive average rankings reproduce scMM 4.88, totalVI 3.63, Seurat v5 2.38, MOFA+ 2.25, SSGATE 1.88, with SSGATE best in ARI and NMI on both noisy datasets
- Each method/dataset combination yields exactly 10 independent replicates (Leiden resolutions 0.1 through 1.0) before metrics are averaged

## Limitations

- Leiden resolution sweeping (0.1-1.0) introduces replicate variability; conclusions depend on the chosen resolution grid
- Ranks are relative to the specific four baselines and two datasets used; the comparison set (e.g., no SpatialGlue) bounds generalizability
- Metric averages across noise levels conflate distinct degradation regimes unless examined per level

## Evidence

- [other] Rank the five methods on each metric per dataset, compute average rankings across all metrics on both datasets, and calculate Robust Rank Aggregation (RRA) p-values.: [evidence span withheld — anchor status: not_found]
- [results] SSGATE achieves the highest average ranking on the BMNC dataset, ranking first in ARI and NMI and second in Purity and HS with an average ranking of 1.5.: [evidence span withheld — anchor status: not_found]
- [results] Under varying levels of Gaussian noise, the comprehensive average rankings are scMM 4.88, totalVI 3.63, Seurat v5 2.38, MOFA+ 2.25, and SSGATE 1.88: [evidence span withheld — anchor status: not_found]
- [other] sweeping the 'resolution' parameter over 0.1, 0.2, …, 1.0 to yield 10 independent experiment replicates per method per dataset: [evidence span withheld — anchor status: not_found]
- [readme] It can also perform self-supervised learning for integration through the graph attention auto-encoders from two paths.: "It can also perform self-supervised learning for integration through the graph attention auto-encoders from two paths."
