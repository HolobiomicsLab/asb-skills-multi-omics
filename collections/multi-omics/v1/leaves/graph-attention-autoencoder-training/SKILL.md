---
name: graph-attention-autoencoder-training
description: Use when you have two preprocessed AnnData objects (transcriptome + proteome)
  each carrying a pruned neighbor network in adata.uns['nbrs_net'] and need a joint
  low-dimensional embedding for clustering or spatially resolved downstream analysis.
when_to_use_negative:
- Input consists of single-omics types other than transcriptome/proteome — the paper
  states the integration of other single-omics data still requires reasonable normalization
  and dimensionality reduction methods not yet available
- Data are not in AnnData format or expression profiles are not stored in anndata.X
  — the README warns the data cannot be read correctly otherwise
- You need a method with systematically derived hyperparameters — the authors note
  defaults are set 'based on experience' rather than derived
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0121
tools:
- name: SSGATE
  role: Core package providing Cal_Nbrs_Net, prune_net, and the ssgate.train dual-path
    graph attention auto-encoder training loop
  repo: github.com/Lv-BioInfo/SSGATE
- name: SCANPY
  role: Preprocessing (scaling/log1p, HVG selection, CLR) and Leiden clustering of
    the resulting integrated embedding
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:31ed983583b8fbcd9c3a2376b21fdd3c60d80796e0a4322fe2e5eb21c7c79689
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClusterProfiler
  - MOFA
  - SCANPY
  - SSGATE
  - Seurat
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# graph-attention-autoencoder-training

## Summary

Train the SSGATE dual-path graph attention auto-encoder to integrate paired transcriptome and proteome data (single-cell or spatially resolved) into a joint 30-dimensional cell embedding via self-supervised triplet loss plus MSE reconstruction loss.

## When to use

Use when you have two preprocessed AnnData objects (transcriptome + proteome) each carrying a pruned neighbor network in adata.uns['nbrs_net'] and need a joint low-dimensional embedding for clustering or spatially resolved downstream analysis.

## When NOT to use

- Input consists of single-omics types other than transcriptome/proteome — the paper states the integration of other single-omics data still requires reasonable normalization and dimensionality reduction methods not yet available
- Data are not in AnnData format or expression profiles are not stored in anndata.X — the README warns the data cannot be read correctly otherwise
- You need a method with systematically derived hyperparameters — the authors note defaults are set 'based on experience' rather than derived

## Inputs

- Transcriptome AnnData object (h5ad) with count-depth scaled log1p expression in anndata.X and top highly variable genes selected
- Proteome AnnData object (h5ad) with centered log-ratio transformed protein expression
- Pruned neighborhood graphs stored in adata.uns['nbrs_net'] from Cal_Nbrs_Net and prune_net (k_cutoff=15, KNN model)
- For spatial data: spatial coordinates in anndata.obsm['spatial']

## Outputs

- Joint 30-dimensional per-cell integrated embedding Z=[Temb|Pemb] suitable for Leiden clustering (resolutions 0.1-1.0)
- Updated AnnData objects returned from ssgate.train with integrated embeddings
- Converged training loss over the 300 epochs

## How to apply

First construct per-omics neighborhood graphs with Cal_Nbrs_Net (feat='X_pca' for expression or 'spatial' for spatial coordinates, KNN model with k_cutoff=15, which Table 2 shows is optimal on BMNC and SLN111_D1) and prune them with prune_net so edges only connect cells in the same Leiden population. Then train with ssgate.train using hidden_dims1=128, hidden_dims2=128, out_dims=30, n_epochs=300, lr=0.001, and sigma=0.1 (the balance parameter lambda weighting the self-supervision terms). Each epoch encodes both modalities through two-layer graph attention encoders (attention on in the first layer, off in the second) and forms the joint representation by concatenation Z=[Temb|Pemb]; the weighted loss is L=(L_recon^t + L_recon^p) + 0.1*(L_super^t + L_super^p) with MSE reconstruction and triplet-loss self-supervision. Stop at the maximum epoch count and export the integrated embedding; validate that it is 30-dimensional per cell, matches the input cell count, and the training loss converges.

## Related tools

- **SSGATE** (Core package providing Cal_Nbrs_Net, prune_net, and the ssgate.train dual-path graph attention auto-encoder training loop) — github.com/Lv-BioInfo/SSGATE
- **SCANPY** (Preprocessing (scaling/log1p, HVG selection, CLR) and Leiden clustering of the resulting integrated embedding)

## Examples

```
adata_st, adata_sp = ssgate.train(adata_st, adata_sp, hidden_dims1=128, hidden_dims2=128, out_dims=30, n_epochs=300, lr=0.001, sigma=0.1)
```

## Evaluation signals

- Output embedding is 30-dimensional per cell and matches the input cell count
- Training loss converges over the 300 epochs under the paper's hyperparameters
- The pruned neighbor network is present in adata.uns['nbrs_net'] for both omics before training
- With k_cutoff=15 and sigma=0.1 fixed, clustering metrics (Purity, Homogeneity, ARI, NMI) across Leiden resolutions 0.1-1.0 reproduce the Table 2 optimum on BMNC and SLN111_D1
- For spatial data (SCS_MT), the UMAP shows well-separated clusters and the spatial map shows strong aggregation with clear boundaries consistent with cortex/medulla anatomy

## Limitations

- Hyperparameters are default settings based on experience rather than systematically derived
- Normalization and dimensionality-reduction methods for integrating omics beyond transcriptome/proteome are not yet available
- For spatial multi-omics without ground-truth labels (e.g. SCS_MT), evaluation is a qualitative acceptance check on spatial aggregation and boundary clarity

## Evidence

- [abstract] self-supervision loss via dual paths: "It can also perform self-supervised learning for integration through the graph attention auto-encoders from two paths."
- [readme] README training hyperparameters: "hidden_dims1 = 128, hidden_dims2 = 128, out_dims = 30, cluster_update_epoch = 20, epochs_init = 50, n_epochs=300, lr=0.001, sigma = 0.1"
- [readme] input format requirement: "The expression profiles are required to be stored in anndata.X, otherwise the data cannot be read correctly"
- [readme] network pruning semantics: "ensuring that each edge 
only connects cells within the"
- [discussion] hyperparameters set by experience: "Based on experience, we provide a set of hyperparameters suitable for most datasets as default settings"
