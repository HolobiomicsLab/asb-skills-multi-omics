---
name: hyperparameter-sensitivity-analysis
description: Use when a fixed hyperparameter of a multi-omics integrator (e.g., SSGATE's
  balance parameter weighting self-reconstruction vs. triplet loss) must be swept
  while others stay constant, to validate defaults via clustering metrics.
when_to_use_negative:
- The goal is benchmarking SSGATE against competitors (MOFA+, TotalVI, ScMM, Seurat
  v5, SpatialGlue) rather than tuning SSGATE's own loss weighting
- Input omics go beyond transcriptome/proteome — the article notes reasonable normalization
  and dimensionality-reduction methods for other single-omics data are not yet available
- No ground-truth labels exist, since the sweep is scored exclusively via label-dependent
  metrics (Purity, Homogeneity Score, ARI, NMI)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0121
tools:
- name: SSGATE
  role: 'Target model: ssgate.train''s sigma (balance parameter of the combined weighted
    loss) is swept over a grid while k_cutoff=15 graphs, hidden_dims 128, out_dims
    30, lr 0.001 and epochs are held fixed'
  repo: https://github.com/Linliu-Bioinf/SSGATE.git
- name: SCANPY
  role: Transcriptome preprocessing (count-depth scaling, log1p, highly-variable-gene
    selection) and Leiden clustering of integrated embeddings across resolutions 0.1–1.0
- name: Seurat v5
  role: Reference network-based normalization workflow replicated when preparing the
    multi-omics inputs for the sensitivity sweep
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:4c9e5ad18deb3c84d42a97b3ed898f3e0c5612f7de88ae53d296d0d9478dadee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Linliu-Bioinf/SSGATE.git
  tools:
  - SCANPY
  - SSGATE
  - Seurat v5
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# hyperparameter-sensitivity-analysis

## Summary

Sweeps a single hyperparameter of SSGATE's combined weighted loss (self-reconstruction + triplet self-supervision) while holding the 15-neighbor graphs and all other settings fixed, then scores Leiden clusterings with Purity, Homogeneity Score, ARI and NMI to justify defaults such as balance=0.1.

## When to use

Use when a fixed hyperparameter of a multi-omics integrator (e.g., SSGATE's balance parameter weighting self-reconstruction vs. triplet loss) must be swept while others stay constant, to validate defaults via clustering metrics.

## When NOT to use

- The goal is benchmarking SSGATE against competitors (MOFA+, TotalVI, ScMM, Seurat v5, SpatialGlue) rather than tuning SSGATE's own loss weighting
- Input omics go beyond transcriptome/proteome — the article notes reasonable normalization and dimensionality-reduction methods for other single-omics data are not yet available
- No ground-truth labels exist, since the sweep is scored exclusively via label-dependent metrics (Purity, Homogeneity Score, ARI, NMI)

## Inputs

- Paired single-cell transcriptome/proteome AnnData objects for BMNC and SLN111_D1 (expression in anndata.X; spatial coordinates in anndata.obsm['spatial'] if applicable)
- Ground-truth cell-type labels per cell for Purity, Homogeneity Score, ARI and NMI computation
- Fixed SSGATE configuration: hidden_dims1=hidden_dims2=128, out_dims=30, lr=0.001, n_epochs=300, KNN graphs with k_cutoff=15
- Balance-parameter (sigma) sweep grid around the default 0.1

## Outputs

- Table 3-style sensitivity table: Purity, Homogeneity Score, ARI, NMI per balance-parameter value for each dataset
- Selected optimal balance parameter (0.1) ranked best in all metrics on both BMNC and SLN111_D1
- Leiden cluster label assignments across resolutions 0.1–1.0 for every sweep setting

## How to apply

Sweep the balance parameter (sigma) of SSGATE's combined weighted loss — self-reconstruction plus triplet self-supervision — around its default of 0.1 while holding everything else fixed: KNN neighbor graphs built per omics with Cal_Nbrs_Net (k_cutoff=15) and pruned by Leiden cluster labels via prune_net, hidden_dims1/2=128, out_dims=30, lr=0.001, 300 epochs. Prepare inputs as in the article: normalize transcriptome profiles by count-depth scaling plus log1p with highly-variable-gene selection (SCANPY / replicated Seurat v5 normalization) and proteome profiles by centered log-ratio transformation, loading BMNC and SLN111_D1 as AnnData with expression in anndata.X. For each sigma setting, cluster the 30-dimensional integrated embeddings with the Leiden algorithm across resolutions 0.1 to 1.0 in 10 independent experiments and compute Purity, Homogeneity Score, ARI, and NMI against ground-truth labels. Aggregate per-setting metric values into a Table-3-style summary per dataset; the sweep is validated when sigma=0.1 gives the best value in all metrics on both datasets. Because the article states its defaults are set 'based on experience', the sweep confirms robustness of the default rather than replacing it with per-dataset tuning.

## Related tools

- **SSGATE** (Target model: ssgate.train's sigma (balance parameter of the combined weighted loss) is swept over a grid while k_cutoff=15 graphs, hidden_dims 128, out_dims 30, lr 0.001 and epochs are held fixed) — https://github.com/Linliu-Bioinf/SSGATE.git
- **SCANPY** (Transcriptome preprocessing (count-depth scaling, log1p, highly-variable-gene selection) and Leiden clustering of integrated embeddings across resolutions 0.1–1.0)
- **Seurat v5** (Reference network-based normalization workflow replicated when preparing the multi-omics inputs for the sensitivity sweep)

## Examples

```
adata = Cal_Nbrs_Net(adata, feat='X_pca', k_cutoff=15, model='KNN'); adata = prune_net(adata); adata_st, adata_sp = ssgate.train(adata_st, adata_sp, out_dims=30, n_epochs=300, lr=0.001, sigma=0.1)
```

## Evaluation signals

- At sigma=0.1 with neighbors fixed at 15, SSGATE ranks best in every metric — Purity, Homogeneity Score, ARI, NMI — on both BMNC and SLN111_D1, matching the reported Table 3
- Each metric value aggregates 10 independent experiments with Leiden resolutions swept 0.1–1.0; per-setting rankings must be stable across replicates
- Only sigma varies across the sweep — k_cutoff=15, hidden_dims=128, out_dims=30, lr=0.001 and 300 epochs must be identical in every run (diff run configs)
- Output table has one row per balance-parameter value and one block per dataset with all four metric columns populated
- Neighbor networks are built per omics with Cal_Nbrs_Net and pruned with prune_net (Leiden-label pruning); skipping pruning invalidates comparability with the paper's setting

## Limitations

- Defaults are chosen 'based on experience'; the sweep validates the default (sigma=0.1) rather than systematically deriving optimal hyperparameters
- Established only on BMNC and SLN111_D1 transcriptome/proteome data; transfer to other tissues, technologies, or omics types is untested
- The README lists n_epochs default 200 while the sensitivity workflow trains 300 epochs — align on epoch count before reproducing Table 3

## Evidence

- [discussion] Hyperparameters are set 'based on experience' as defaults rather than systematically derived: "Based on experience, we provide a set of hyperparameters suitable for most datasets as default settings"
- [readme] Neighbor count is controlled by the KNN k_cutoff parameter of Cal_Nbrs_Net, held at 15 during the sweep: "- `k_cutoff`: Defines the number of neighbors when using the KNN model. This value is required if the KNN model is used."
- [readme] The embedding dimensionality (out_dims=30) held fixed throughout the sensitivity sweep: "The dimensionality of the output layer. This is the size of the embedding space where the integrated features from both datasets will be projected."
- [abstract] The swept balance parameter weights the self-supervision (triplet) loss term added to self-reconstruction: "It can also perform self-supervised learning for integration through the graph attention auto-encoders from two paths."
