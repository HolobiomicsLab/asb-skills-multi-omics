---
name: clustering-metric-computation-ari-nmi-purity-homogeneity
description: Use when comparing integration methods (e.g., SSGATE, MOFA+, Seurat v5,
  totalVI, scMM) on paired transcriptome-proteome single-cell data via Leiden clustering,
  or when reproducing noise-robustness rankings under degraded data quality.
when_to_use_negative:
- Data is spatially resolved multi-omics where spatial coordinates (anndata.obsm['spatial'])
  drive graph construction — this skill is for single-cell transcriptome-proteome
  benchmarking
- No ground-truth cell labels exist, so ARI/NMI/Purity/Homogeneity cannot be computed
- Goal is evaluating spatial domain detection rather than noise robustness of single-cell
  integration methods
edam_operation: http://edamontology.org/operation_0558
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: SCANPY
  role: Leiden clustering of integration embeddings across resolutions 0.1–1.0 for
    each replicate
- name: Leiden algorithm
  role: Community-detection clustering applied at each resolution on the SSGATE and
    baseline embeddings
- name: SSGATE
  role: Method under test, run with defaults (15 neighbors, balance 0.1, embedding
    dim 30, 300 epochs, lr 0.001) on noisy inputs
  repo: https://github.com/Linliu-Bioinf/SSGATE
- name: MOFA+
  role: Baseline integration method whose clustering metrics contribute to the ranking
- name: Seurat v5
  role: Baseline integration method whose clustering metrics contribute to the ranking
- name: TotalVI
  role: Baseline integration method whose clustering metrics contribute to the ranking
- name: scMM
  role: Baseline integration method whose clustering metrics contribute to the ranking
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:e105ee8d8e4e58173d9c26b72d8deefa59f37e34d63b7a06fcf6828bc6af4dff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Linliu-Bioinf/SSGATE
  tools:
  - Leiden algorithm
  - MOFA
  - MOFA+
  - SCANPY
  - SSGATE
  - ScMM
  - Seurat v5
  - TotalVI
  - scMM
  edam_operation: http://edamontology.org/operation_0558
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# clustering-metric-computation-ari-nmi-purity-homogeneity

## Summary

Benchmark protocol for scoring multi-omics integration embeddings by clustering agreement with ground-truth cell labels, using Purity, Homogeneity Score, Adjusted Rand Index (ARI), and Normalized Mutual Information (NMI) as implemented in the SSGATE robustness study.

## When to use

Use when comparing integration methods (e.g., SSGATE, MOFA+, Seurat v5, totalVI, scMM) on paired transcriptome-proteome single-cell data via Leiden clustering, or when reproducing noise-robustness rankings under degraded data quality.

## When NOT to use

- Data is spatially resolved multi-omics where spatial coordinates (anndata.obsm['spatial']) drive graph construction — this skill is for single-cell transcriptome-proteome benchmarking
- No ground-truth cell labels exist, so ARI/NMI/Purity/Homogeneity cannot be computed
- Goal is evaluating spatial domain detection rather than noise robustness of single-cell integration methods

## Inputs

- Integrated cell embeddings (dimension 30 for SSGATE) from each method applied to noisy paired transcriptome-proteome h5ad datasets (BMNC, SLN111_D1)
- Ground-truth cell-type labels for each dataset
- Gaussian-noise-augmented versions of each dataset at three noise levels

## Outputs

- Per-replicate, per-resolution table of Purity, Homogeneity Score, ARI, and NMI vs. ground-truth labels
- Comprehensive average ranking per method averaged across three noise levels (expected: SSGATE 1.88, MOFA+ 2.25, Seurat v5 2.38, totalVI 3.63, scMM 4.88)
- noise_robustness_results.csv

## How to apply

Cluster each method's integrated embedding with the Leiden algorithm in SCANPY across resolutions 0.1, 0.2, ..., 1.0, then compute Purity, Homogeneity Score, Adjusted Rand Index, and Normalized Mutual Information against ground-truth labels for every method, noise level, and replicate (10 independent replicates per method). For noise-robustness experiments, inject three levels of Gaussian noise into each paired transcriptome-proteome h5ad dataset (e.g., BMNC and SLN111_D1 from the SSGATE repository), run each method on the noisy inputs, and average metric rankings across the three noise levels into a comprehensive average ranking. Treat lower average rank as better — in the SSGATE paper the reported comprehensive average rankings were SSGATE 1.88, MOFA+ 2.25, Seurat v5 2.38, totalVI 3.63, and scMM 4.88. To claim robustness for a method, it should achieve the best ARI and NMI on both noisy datasets, matching the paper's Fig. 3 pattern; write all per-method, per-noise-level, per-replicate metrics to a results CSV.

## Related tools

- **SCANPY** (Leiden clustering of integration embeddings across resolutions 0.1–1.0 for each replicate)
- **Leiden algorithm** (Community-detection clustering applied at each resolution on the SSGATE and baseline embeddings)
- **SSGATE** (Method under test, run with defaults (15 neighbors, balance 0.1, embedding dim 30, 300 epochs, lr 0.001) on noisy inputs) — https://github.com/Linliu-Bioinf/SSGATE
- **MOFA+** (Baseline integration method whose clustering metrics contribute to the ranking)
- **Seurat v5** (Baseline integration method whose clustering metrics contribute to the ranking)
- **TotalVI** (Baseline integration method whose clustering metrics contribute to the ranking)
- **scMM** (Baseline integration method whose clustering metrics contribute to the ranking)

## Examples

```
sc.tl.leiden(adata, resolution=0.1); ari = adjusted_rand_score(adata.obs['ground_truth'], adata.obs['leiden']); nmi = normalized_mutual_info_score(adata.obs['ground_truth'], adata.obs['leiden'])
```

## Evaluation signals

- Comprehensive average rankings match the reported values: scMM 4.88, totalVI 3.63, Seurat v5 2.38, MOFA+ 2.25, SSGATE 1.88 (lower is better)
- SSGATE attains the best ARI and NMI on both noisy datasets (BMNC and SLN111_D1), consistent with Fig. 3
- Ten independent replicates per method are obtained by sweeping Leiden resolution over 0.1, 0.2, ..., 1.0
- Metrics are averaged across all three Gaussian-noise levels before computing ranks
- All metric rows are present in noise_robustness_results.csv for every method × noise level × replicate combination

## Limitations

- Baseline method software versions are not reported in the paper, so exact numeric reproduction across environments is uncertain
- Hyperparameters (e.g., SSGATE's defaults: 15 neighbors, balance 0.1, dim 30, 300 epochs, lr 0.001) are set based on experience rather than systematically derived
- Protocol is designed for transcriptome-proteome pairs; integrating other omics types requires additional normalization/dimensionality-reduction choices

## Evidence

- [results] Under varying levels of Gaussian noise, the comprehensive average rankings are scMM 4.88, totalVI 3.63, Seurat v5 2.38, MOFA+ 2.25, and SSGATE 1.88: [evidence span withheld — anchor status: not_found]
- [readme] It can also perform self-supervised learning for integration through the graph attention auto-encoders from two paths.: "It can also perform self-supervised learning for integration through the graph attention auto-encoders from two paths."
- [abstract] SSGATE shows better performance and stronger robustness than competitive methods and facilitates downstream analysis.: "SSGATE shows better performance and stronger robustness than competitive methods and facilitates downstream analysis."
