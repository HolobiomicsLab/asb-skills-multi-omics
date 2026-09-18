---
name: clustering-metric-benchmarking
description: Use when decision support for tuning the number of neighbors (k_cutoff)
  used in SSGATE's Cal_Nbrs_Net graph construction on single-cell multi-omics (h5ad)
  data, where the goal is to reproduce or validate the hyperparameter sensitivity
  table (Table 2).
when_to_use_negative:
- Data is spatially resolved multi-omics where graph construction should use spatial
  coordinates (obsm['spatial']) with the Radius model rather than expression-based
  KNN neighbor sweeps
- The question concerns other hyperparameters (e.g. hidden dimensions, epochs) rather
  than the neighbor count sweep
- Data formats or omics modalities beyond transcriptome/proteome, since SSGATE normalization
  and dimensionality reduction for other single-omics data is not yet available
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCANPY
  role: 'Preprocessing of h5ad inputs: count-depth scaling with log1p, highly variable
    gene selection, PCA, and CLR normalization for proteomics'
- name: SSGATE
  role: Provides Cal_Nbrs_Net and prune_net for KNN graph construction/pruning per
    k_cutoff value, and the dual-path graph attention auto-encoder trained with sigma=0.1,
    300 epochs, out_dims=30, lr=0.001
  repo: https://github.com/Linliu-Bioinf/SSGATE.git
- name: Leiden
  role: Clustering of integrated embeddings at resolutions 0.1 through 1.0; also used
    in prune_net for neighbor pruning
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:190a8fe9fd7abc4141950a52c73c3faf7ed25c1c94d75a3e983d0616dd487b32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Linliu-Bioinf/SSGATE
  tools:
  - Leiden
  - SCANPY
  - SSGATE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# clustering-metric-benchmarking

## Summary

Benchmark hyperparameter sensitivity of SSGATE's neighborhood-graph construction by sweeping the KNN neighbor count (k_cutoff), retraining the dual-path graph attention auto-encoder, and comparing Leiden clustering quality across Purity, Homogeneity, ARI, and NMI to identify the optimal neighbor setting.

## When to use

Decision support for tuning the number of neighbors (k_cutoff) used in SSGATE's Cal_Nbrs_Net graph construction on single-cell multi-omics (h5ad) data, where the goal is to reproduce or validate the hyperparameter sensitivity table (Table 2).

## When NOT to use

- Data is spatially resolved multi-omics where graph construction should use spatial coordinates (obsm['spatial']) with the Radius model rather than expression-based KNN neighbor sweeps
- The question concerns other hyperparameters (e.g. hidden dimensions, epochs) rather than the neighbor count sweep
- Data formats or omics modalities beyond transcriptome/proteome, since SSGATE normalization and dimensionality reduction for other single-omics data is not yet available

## Inputs

- Single-cell multi-omics datasets BMNC and SLN111_D1 in AnnData (.h5ad) format with expression in anndata.X
- Ground-truth cell type labels for metric computation
- SCANPY-preprocessed matrices: log1p-scaled HVG transcriptomics (200 PCs) and CLR-normalized proteomics

## Outputs

- Hyperparameter sensitivity table (Table 2 equivalent) of Purity, Homogeneity Score, ARI, and NMI per neighbor count and Leiden resolution
- Identified optimal neighbor count (k_cutoff=15) per dataset

## How to apply

Load the single-cell multi-omics datasets BMNC and SLN111_D1 (h5ad) and preprocess with SCANPY: count-depth scaling with log1p plus highly variable gene selection for transcriptomics, and centered log-ratio transformation for proteomics. Reduce the transcriptome to 200 principal components with PCA, then for each k_cutoff value in the sweep build a KNN neighborhood graph per omics modality via the SSGATE Cal_Nbrs_Net function (model='KNN') and prune edges with prune_net using Leiden-based neighbor pruning. Train the dual-path graph attention auto-encoder with the balance parameter sigma fixed at 0.1, 300 epochs, embedding dimension out_dims=30, and learning rate lr=0.001. Cluster each integrated embedding with the Leiden algorithm at resolutions 0.1 through 1.0 (10 independent settings) on both datasets, compute Purity, Homogeneity Score, Adjusted Rand Index, and Normalized Mutual Information against ground-truth labels, and aggregate per neighbor setting. Correct execution is confirmed when k_cutoff=15 yields optimal values in all four metrics on both BMNC and SLN111_D1, matching the paper's Table 2.

## Related tools

- **SCANPY** (Preprocessing of h5ad inputs: count-depth scaling with log1p, highly variable gene selection, PCA, and CLR normalization for proteomics)
- **SSGATE** (Provides Cal_Nbrs_Net and prune_net for KNN graph construction/pruning per k_cutoff value, and the dual-path graph attention auto-encoder trained with sigma=0.1, 300 epochs, out_dims=30, lr=0.001) — https://github.com/Linliu-Bioinf/SSGATE.git
- **Leiden** (Clustering of integrated embeddings at resolutions 0.1 through 1.0; also used in prune_net for neighbor pruning)

## Examples

```
adata = Cal_Nbrs_Net(adata, feat='X_pca', k_cutoff=15, model='KNN'); adata = prune_net(adata); adata_st, adata_sp = ssgate.train(adata_st, adata_sp, out_dims=30, n_epochs=300, lr=0.001, sigma=0.1)
```

## Evaluation signals

- With sigma fixed at 0.1, all four metrics (Purity, Homogeneity, ARI, NMI) peak at 15 neighbors on both BMNC and SLN111_D1
- The sweep covers 10 Leiden resolutions (0.1–1.0) per neighbor setting, and results are aggregated per setting into the output table
- Neighborhood networks are stored in adata.uns['nbrs_net'] after Cal_Nbrs_Net and updated after prune_net
- Embedding dimension of the trained model is 30 (out_dims=30) and training reaches the 300-epoch budget with lr=0.001
- Transcriptomics uses 200 principal components and proteomics uses centered log-ratio transformation before graph construction

## Limitations

- Hyperparameters are set 'based on experience' as defaults rather than systematically derived
- The result is dataset-specific: optimality at 15 neighbors was verified only on BMNC and SLN111_D1; other datasets may require a different k_cutoff
- Integration of omics beyond transcriptome/proteome lacks established normalization and dimensionality reduction methods

## Evidence

- [readme] k_cutoff defines number of neighbors: "`k_cutoff`: Defines the number of neighbors when using the KNN model. This value is required if the KNN model is used."
- [readme] training defaults for sweep: "n_epochs=300, 
                                      lr=0.001,
                                      sigma = 0.1)"
- [readme] pruning rationale: "This function is used to prune the adjacency network calculated by the `Cal_Nbrs_Net` function, ensuring that each edge"
- [discussion] defaults set by experience: "Based on experience, we provide a set of hyperparameters suitable for most datasets as default settings"
- [readme] output of training: "After multiple rounds of training, the joint feature 
representation of the two omics can be obtained."
