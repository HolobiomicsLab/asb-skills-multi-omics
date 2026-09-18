---
name: single-cell-multiomics-integration-training
description: Use when paired transcriptome–proteome data are available as AnnData
  (.h5ad) objects and a joint low-dimensional embedding is needed for Leiden clustering
  and metric-based comparison against methods like MOFA+, Seurat v5, totalVI, or scMM.
when_to_use_negative:
- The omics data are not transcriptome/proteome — the article states normalization
  and dimensionality-reduction methods for integrating other single-omics data are
  not yet available
- The expression profiles are not stored in anndata.X — the README warns 'the data
  cannot be read correctly' otherwise
- Only a feature table already containing an integrated representation is available
  and no paired multi-omics raw profiles exist to build per-modality neighborhood
  graphs
tools:
- name: SSGATE
  role: 'Primary method: Cal_Nbrs_Net neighborhood-graph construction, prune_net edge
    pruning, and ssgate.train dual-path graph attention auto-encoder training'
  repo: https://github.com/Linliu-Bioinf/SSGATE
- name: SCANPY
  role: Preprocessing of RNA (count-depth scaling + log1p, HVG selection) and Leiden
    clustering of the integrated embedding across resolutions 0.1–1.0
- name: Seurat v5
  role: Reference for replicated normalization of transcriptome profiles in the source-card
    workflows; also the WNN-graph baseline SSGATE is benchmarked against
merged_aliases:
- single-cell-multiomics-integration-benchmarking
- single-cell-multiomics-integration-method-benchmarking
merged_alias_records:
- alias: single-cell-multiomics-integration-benchmarking
  slug: single-cell-multiomics-integration-benchmarking
  jaccard_score: 0.6667
  method: token-set-jaccard
  decision: needs_review
- alias: single-cell-multiomics-integration-method-benchmarking
  slug: single-cell-multiomics-integration-method-benchmarking
  jaccard_score: 0.5714
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_004
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:7ada39ae772212023a4c924f5fa1169eb5f6c9f231306a9db41b062860cd7dba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Linliu-Bioinf/SSGATE
  tools:
  - MOFA
  - SCANPY
  - SSGATE
  - ScMM
  - Seurat v5
  - TotalVI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# single-cell-multiomics-integration-training

## Summary

Train SSGATE, a dual-path graph attention auto-encoder, to integrate paired transcriptome–proteome single-cell data by building pruned neighborhood graphs per omics modality and learning a joint 30-dimensional embedding via self-supervised learning. It underpins the paper's benchmark, noise-robustness, and hyperparameter-sensitivity analyses on the BMNC and SLN111_D1 datasets.

## When to use

Use when paired transcriptome–proteome data are available as AnnData (.h5ad) objects and a joint low-dimensional embedding is needed for Leiden clustering and metric-based comparison against methods like MOFA+, Seurat v5, totalVI, or scMM.

## When NOT to use

- The omics data are not transcriptome/proteome — the article states normalization and dimensionality-reduction methods for integrating other single-omics data are not yet available
- The expression profiles are not stored in anndata.X — the README warns 'the data cannot be read correctly' otherwise
- Only a feature table already containing an integrated representation is available and no paired multi-omics raw profiles exist to build per-modality neighborhood graphs

## Inputs

- Paired transcriptome–proteome AnnData (.h5ad) objects (e.g., BMNC, SLN111_D1 from the BGI cloud disk linked in the SSGATE repository)
- RNA expression stored in anndata.X with log1p normalization and highly variable gene selection
- Protein expression stored in anndata.X with centered log-ratio (CLR) normalization
- Optional spatial coordinates in anndata.obsm['spatial'] for spatially resolved inputs

## Outputs

- Joint 30-dimensional integrated embedding (out_dims=30) returned as AnnData objects (adata_st, adata_sp)
- Neighbor network stored in adata.uns['nbrs_net'] after Cal_Nbrs_Net and prune_net
- Trained SSGATE model state usable for Leiden clustering and metric computation (Purity, HS, ARI, NMI)

## How to apply

Load the paired transcriptome–proteome h5ad inputs and preprocess in SCANPY: count-depth scaling with log1p transformation plus highly variable gene selection for RNA, and centered log-ratio (CLR) transformation for protein. For each omics separately, call Cal_Nbrs_Net to build a neighbor network (KNN model with k_cutoff=15 as used in the benchmark, or Radius model with rad_cutoff; features from 'X_pca' for single-cell data or 'spatial' coordinates for spatial data), then apply prune_net so edges connect only cells of the same cell population. Train with ssgate.train using the benchmark defaults: hidden_dims1=hidden_dims2=128, out_dims=30 (embedding dimension), n_epochs=300, lr=0.001, and the combined weighted loss balance parameter (self-reconstruction vs. self-supervised triplet loss) set to 0.1, which Table 3 shows is optimal with 15 neighbors on both BMNC and SLN111_D1. The returned AnnData objects carry the joint feature representation used for downstream Leiden clustering (resolutions 0.1–1.0, 10 replicates) and scoring with Purity, Homogeneity Score, ARI, and NMI.

## Related tools

- **SSGATE** (Primary method: Cal_Nbrs_Net neighborhood-graph construction, prune_net edge pruning, and ssgate.train dual-path graph attention auto-encoder training) — https://github.com/Linliu-Bioinf/SSGATE
- **SCANPY** (Preprocessing of RNA (count-depth scaling + log1p, HVG selection) and Leiden clustering of the integrated embedding across resolutions 0.1–1.0)
- **Seurat v5** (Reference for replicated normalization of transcriptome profiles in the source-card workflows; also the WNN-graph baseline SSGATE is benchmarked against)

## Examples

```
adata = Cal_Nbrs_Net(adata, feat='X_pca', k_cutoff=15, model='KNN'); adata = prune_net(adata); adata_st, adata_sp = ssgate.train(adata_st, adata_sp, out_dims=30, n_epochs=300, lr=0.001, sigma=0.1)
```

## Evaluation signals

- Neighbor network is stored in adata.uns['nbrs_net'] after Cal_Nbrs_Net and survives prune_net with edges restricted to same-population cells
- Output embedding has the expected dimensionality (out_dims=30) and training completes the full 300 epochs at lr=0.001
- Balance-parameter sweep with neighbors fixed at 15 reproduces Table 3: best results in all metrics (Purity, HS, ARI, NMI) on both BMNC and SLN111_D1 at balance parameter 0.1
- Benchmark validation: SSGATE attains average ranking 1.5 on BMNC (first in ARI and NMI) and 2.25 on SLN111_D1 (tied with totalVI), with the smallest RRA p-value among the five methods
- Noise-robustness check: comprehensive average rankings match scMM 4.88, totalVI 3.63, Seurat v5 2.38, MOFA+ 2.25, SSGATE 1.88, with SSGATE best in ARI and NMI on noisy data

## Limitations

- Hyperparameters are provided 'based on experience' as defaults rather than systematically derived; note the two READMEs disagree on the n_epochs default (300 in the Linliu-Bioinf README train example vs. 200 in the Lv-BioInfo parameter table), so set n_epochs=300 explicitly to match the paper
- Spatial coordinates must be stored in anndata.obsm['spatial']; if missing or misplaced, the data are silently processed as single-cell multi-omics
- No software versions are reported for the compared methods, and normalization/dimensionality reduction for omics types beyond transcriptome/proteome is deferred to future work
- Benchmarked datasets are distributed only via a password-protected BGI cloud disk link, which may affect reproducibility

## Evidence

- [readme] readme: "It can construct the neighborhood graphs based on single-cell expression profiles or spatial coordinates, enabling it to process single-cell data and utilize spatial information"
- [intro] intro: "This study presents a multi-omics integration method based on dual-path GATE (SSGATE) for both single-cell and spatially resolved data."
- [discussion] discussion: "Based on experience, we provide a set of hyperparameters suitable for most datasets as default settings"
