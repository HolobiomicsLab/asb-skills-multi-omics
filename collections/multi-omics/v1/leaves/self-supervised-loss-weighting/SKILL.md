---
name: self-supervised-loss-weighting
description: Use when tuning SSGATE's combined weighted loss on paired transcriptome-proteome
  data (e.g. BMNC, SLN111_D1) and you need to determine how the balance parameter
  affects clustering/integration performance at a fixed neighbor count (15).
when_to_use_negative:
- The goal is to compare SSGATE against competing methods (MOFA+, Seurat v5, TotalVI,
  etc.) rather than tune its loss weighting
- The task involves omics types beyond transcriptome/proteome, for which SSGATE normalization
  and dimensionality reduction are not yet available
- You need systematically derived hyperparameters — the paper itself sets defaults
  'based on experience' rather than deriving them
tools:
- name: SSGATE
  role: Trains the dual-path graph attention auto-encoder; the sigma parameter of
    ssgate.train() controls the balance-parameter sweep of the combined self-reconstruction/self-supervision
    (triplet) loss
  repo: https://github.com/Lv-BioInfo/SSGATE
- name: SCANPY
  role: Used for Leiden clustering of integrated embeddings across resolutions 0.1-1.0
    and for transcriptome normalization (count depth scaling plus log1p) prior to
    graph construction
- name: Seurat v5
  role: Reference for replicated Seurat-style normalization of the input transcriptome/proteome
    profiles
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:04ef23fa60937212705c1a02a40030aaa765f39491e4d20ac9911ff2a365edef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lv-BioInfo/SSGATE
  tools:
  - SCANPY
  - SSGATE
  - Seurat v5
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# self-supervised-loss-weighting

## Summary

Sweep the balance parameter (sigma) that weights the self-supervised triplet loss against self-reconstruction in SSGATE's combined loss to find the setting that maximizes integration quality, following the paper's Table 3 protocol.

## When to use

Use when tuning SSGATE's combined weighted loss on paired transcriptome-proteome data (e.g. BMNC, SLN111_D1) and you need to determine how the balance parameter affects clustering/integration performance at a fixed neighbor count (15).

## When NOT to use

- The goal is to compare SSGATE against competing methods (MOFA+, Seurat v5, TotalVI, etc.) rather than tune its loss weighting
- The task involves omics types beyond transcriptome/proteome, for which SSGATE normalization and dimensionality reduction are not yet available
- You need systematically derived hyperparameters — the paper itself sets defaults 'based on experience' rather than deriving them

## Inputs

- Normalized transcriptome and proteome AnnData objects (expression in anndata.X) for BMNC and SLN111_D1
- Neighbor networks stored in adata.uns['nbrs_net'] with k_cutoff=15 (KNN model)
- Ground-truth cell-type labels for computing Purity, Homogeneity, ARI, and NMI

## Outputs

- Integrated joint embeddings (out_dims=30) per balance-parameter setting
- Leiden cluster assignments across resolutions 0.1-1.0 (10 independent runs per setting)
- A hyperparameter sensitivity summary table (Table 3 analogue) of Purity, Homogeneity Score, ARI, and NMI per sigma value per dataset

## How to apply

Fix the number of neighbors at 15 and construct neighbor networks for both transcriptome and proteome AnnData objects with Cal_Nbrs_Net, then prune with prune_net. Train the dual-path graph attention auto-encoder while varying sigma (the balance parameter in the combined weighted loss of self-reconstruction and self-supervision triplet losses), sweeping around the default and confirming at 0.1, using out_dims=30, lr=0.001, and up to 300 epochs per the README's train signature. For each sigma setting, cluster the integrated embeddings with the Leiden algorithm across resolutions 0.1 to 1.0 over 10 independent experiments. Compute Purity, Homogeneity Score, ARI, and NMI against ground-truth labels and aggregate per-setting rankings into a Table 3-style summary for both datasets. Validation: the sweep is correct if sigma=0.1 yields the best results in all metrics on both datasets, consistent with the paper's reported Table 3.

## Related tools

- **SSGATE** (Trains the dual-path graph attention auto-encoder; the sigma parameter of ssgate.train() controls the balance-parameter sweep of the combined self-reconstruction/self-supervision (triplet) loss) — https://github.com/Lv-BioInfo/SSGATE
- **SCANPY** (Used for Leiden clustering of integrated embeddings across resolutions 0.1-1.0 and for transcriptome normalization (count depth scaling plus log1p) prior to graph construction)
- **Seurat v5** (Reference for replicated Seurat-style normalization of the input transcriptome/proteome profiles)

## Examples

```
adata_st, adata_sp = ssgate.train(adata_st, adata_sp, hidden_dims1=128, hidden_dims2=128, out_dims=30, cluster_update_epoch=20, epochs_init=50, n_epochs=300, lr=0.001, sigma=0.1)
```

## Evaluation signals

- With neighbors fixed at 15, SSGATE should obtain the best results in ALL metrics (Purity, Homogeneity, ARI, NMI) on both BMNC and SLN111_D1 at sigma=0.1
- Each metric is computed over 10 independent experiments with Leiden resolutions swept from 0.1 to 1.0 — rankings should be aggregated before judging
- Training must use the README defaults: hidden_dims1=hidden_dims2=128, out_dims=30, lr=0.001, n_epochs=300
- Neighbor graphs must be built with k_cutoff=15 and pruned with prune_net before training; sigma is the only factor varied
- Per-setting metric tables should reproduce the ordering reported in Table 3 for both datasets

## Limitations

- Hyperparameters are provided 'based on experience' as defaults suitable for most datasets, not systematically derived; the optimal sigma may differ on other tissues or sequencing technologies
- The README is internally inconsistent on epochs (train example shows n_epochs=300 while the parameter description states a default of 200), so confirm the intended value before reproducing
- Results are reported for transcriptome-proteome integration only; sigma transferability to other omics combinations is untested

## Evidence

- [results] sigma balance finding: [evidence span withheld — anchor status: not_found]
- [readme] README training call: "After completing the above two steps, training can be carried out. After multiple rounds of training, the joint feature representation of the two omics can be obtained."
- [discussion] default hyperparameters caveat: "Based on experience, we provide a set of hyperparameters suitable for most datasets as default settings"
- [abstract] self-supervised dual-path learning: "It can also perform self-supervised learning for integration through the graph attention auto-encoders from two paths."
