---
name: clustering-agreement-metric-computation
description: Use when you have integrated embeddings from one or more multi-omics
  methods (e.g. SSGATE, MOFA+, Seurat v5, totalVI, scMM) with known ground-truth cell
  labels and need a reproducible, multi-metric comparison of clustering agreement.
when_to_use_negative:
- The embeddings lack ground-truth cell-type annotations — Purity, HS, ARI, and NMI
  all require reference labels
- You need to evaluate the integrated embedding itself (e.g. spatial-coordinate-aware
  neighborhood construction) rather than the downstream clustering agreement
- Only a single clustering run at one resolution is available — the skill's robustness
  claim depends on the 10-replicate resolution sweep
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0091
tools:
- name: SSGATE
  role: Primary integration method whose dual-path graph attention auto-encoder embeddings
    are clustered and scored; expected to attain an average ranking of 1.5 on BMNC
    and 2.25 on SLN111_D1
  repo: https://github.com/Linliu-Bioinf/SSGATE
- name: SCANPY
  role: Preprocessing (log1p scaling, HVG selection, CLR for protein) and Leiden clustering
    of the integrated embeddings across the 0.1-1.0 resolution sweep
- name: MOFA+
  role: Statistical baseline method producing embeddings benchmarked by the same metric
    pipeline
- name: Seurat v5
  role: Network-based baseline using a weighted nearest neighbor graph, benchmarked
    by the same metric pipeline
- name: TotalVI
  role: Probabilistic baseline (scVI-like joint representation of gene expression
    and protein) benchmarked by the same metric pipeline; tied with SSGATE at 2.25
    average ranking on SLN111_D1
- name: ScMM
  role: Mixture-of-experts multimodal VAE baseline benchmarked by the same metric
    pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:6d096d9f54a7b17e70882b7d247747c02b9cb686629deed61012d1d1565c78bf
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# clustering-agreement-metric-computation

## Summary

Benchmark clustering quality by comparing Leiden cluster labels of integrated single-cell/spatial multi-omics embeddings against ground-truth cell-type labels using four agreement metrics — Purity (P), Homogeneity Score (HS), Adjusted Rand Index (ARI), and Normalized Mutual Information (NMI) — then rank methods per metric and aggregate with Robust Rank Aggregation (RRA) p-values.

## When to use

Use when you have integrated embeddings from one or more multi-omics methods (e.g. SSGATE, MOFA+, Seurat v5, totalVI, scMM) with known ground-truth cell labels and need a reproducible, multi-metric comparison of clustering agreement.

## When NOT to use

- The embeddings lack ground-truth cell-type annotations — Purity, HS, ARI, and NMI all require reference labels
- You need to evaluate the integrated embedding itself (e.g. spatial-coordinate-aware neighborhood construction) rather than the downstream clustering agreement
- Only a single clustering run at one resolution is available — the skill's robustness claim depends on the 10-replicate resolution sweep

## Inputs

- Integrated cell embeddings for each method, stored in AnnData (h5ad) format (e.g. BMNC and SLN111_D1 paired transcriptome-proteome datasets preprocessed in SCANPY with log1p/HVG for RNA and CLR for protein)
- Ground-truth cell-type label vector per dataset
- Method-per-dataset embedding files from each compared tool (SSGATE, MOFA+, Seurat v5, totalVI, scMM)

## Outputs

- Full metrics table (benchmark_metrics.csv) with Purity, HS, ARI, NMI for every method/dataset/resolution combination
- Average rankings across all metrics on both datasets per method
- Robust Rank Aggregation (RRA) p-values per method

## How to apply

Cluster each integrated embedding (stored in the AnnData embedding space) with the Leiden algorithm, sweeping the resolution parameter over 0.1, 0.2, ..., 1.0 to yield 10 independent replicates per method per dataset (e.g. BMNC and SLN111_D1). For every method/dataset/resolution combination, compute Purity, Homogeneity Score, Adjusted Rank Index (ARI), and Normalized Mutual Information (NMI) against the ground-truth labels. Rank the competing methods (SSGATE vs. MOFA+, Seurat v5 weighted-nearest-neighbor, totalVI, scMM mixture-of-experts VAE) on each metric per dataset, compute average rankings across all metrics on both datasets, and calculate Robust Rank Aggregation p-values. Save the full metrics table and the average rankings plus RRA p-values to benchmark_metrics.csv. The rationale for the resolution sweep is robustness: averaging over 10 clustering replicates prevents a single favorable Leiden resolution from driving a method's rank.

## Related tools

- **SSGATE** (Primary integration method whose dual-path graph attention auto-encoder embeddings are clustered and scored; expected to attain an average ranking of 1.5 on BMNC and 2.25 on SLN111_D1) — https://github.com/Linliu-Bioinf/SSGATE
- **SCANPY** (Preprocessing (log1p scaling, HVG selection, CLR for protein) and Leiden clustering of the integrated embeddings across the 0.1-1.0 resolution sweep)
- **MOFA+** (Statistical baseline method producing embeddings benchmarked by the same metric pipeline)
- **Seurat v5** (Network-based baseline using a weighted nearest neighbor graph, benchmarked by the same metric pipeline)
- **TotalVI** (Probabilistic baseline (scVI-like joint representation of gene expression and protein) benchmarked by the same metric pipeline; tied with SSGATE at 2.25 average ranking on SLN111_D1)
- **ScMM** (Mixture-of-experts multimodal VAE baseline benchmarked by the same metric pipeline)

## Examples

```
for r in [0.1,0.2,0.3,0.4,0.5,0.6,0.7,0.8,0.9,1.0]: sc.tl.leiden(adata, resolution=r, key_added=f'leiden_{r}'); metrics.append(compute_agreement(adata.obs[f'leiden_{r}'], adata.obs['truth']))
```

## Evaluation signals

- Metrics computed for every method/dataset/resolution combination (5 methods x 2 datasets x 10 resolutions = 100 rows expected in benchmark_metrics.csv)
- SSGATE should rank first in ARI and NMI and second in Purity and HS on BMNC, giving an average ranking of 1.5
- On SLN111_D1, SSGATE should reach an average ranking of 2.25, tied with totalVI
- SSGATE should show the smallest Robust Rank Aggregation p-value among the five methods, matching the reported Fig. 2 results
- Every metric value must fall in [0,1] and ARI must not exceed its NMI/HS counterparts' plausibility bounds; ARI can be negative for random labelings

## Limitations

- Hyperparameters are set 'based on experience' as defaults rather than systematically derived, so benchmarked embeddings may not reflect each method's best possible tuning
- No software versions are reported for the compared methods (e.g. no version for scMM, totalVI, MOFA+), limiting reproducibility
- Benchmarked datasets are referred to only generically in the discussion; the BMNC/SLN111_D1 h5ad files are only available via a BGI cloud-disk link with an extract password, not a stable accession

## Evidence

- [other] Cluster every integrated embedding with the Leiden algorithm, sweeping the 'resolution' parameter over 0.1, 0.2, …, 1.0 to yield 10 independent experiment replicates per method per dataset.: [evidence span withheld — anchor status: not_found]
- [other] Compare cluster labels to ground truth by computing Purity (P), Homogeneity Score (HS), Adjusted Rand Index (ARI), and Normalized Mutual Information (NMI).: [evidence span withheld — anchor status: not_found]
- [abstract] SSGATE shows better performance and stronger robustness than competitive methods and facilitates downstream analysis.: "SSGATE shows better performance and stronger robustness than competitive methods and facilitates downstream analysis."
- [readme] The expression profiles are required to be stored in anndata.X, otherwise the data cannot be read correctly.: "The expression profiles are required to be stored in anndata.X, otherwise the data cannot be read correctly."
- [discussion] Based on experience, we provide a set of hyperparameters suitable for most datasets as default settings: "Based on experience, we provide a set of hyperparameters suitable for most datasets as default settings"
