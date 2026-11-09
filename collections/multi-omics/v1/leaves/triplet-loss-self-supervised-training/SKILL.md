---
name: triplet-loss-self-supervised-training
description: Use when integrating paired transcriptome and proteome data (single-cell
  or spatial multi-omics) in AnnData format into a joint low-dimensional embedding
  via self-supervised dual-path graph attention auto-encoder training with triplet
  loss.
when_to_use_negative:
- Integrating omics modalities other than transcriptomics/proteomics — the paper notes
  that normalization and dimensionality reduction methods for other single-omics data
  are not yet available
- Data that has not been processed by Cal_Nbrs_Net and prune_net — the neighborhood
  network construction and pruning steps must precede training
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SSGATE
  role: Provides the dual-path graph attention auto-encoder and the ssgate.train routine
    implementing triplet-loss self-supervision plus MSE reconstruction training
  repo: https://github.com/Linliu-Bioinfo/SSGATE.git
- name: anndata
  role: 'Required input container format: expression profiles must be stored in anndata.X
    and spatial coordinates in anndata.obsm[''spatial'']'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:f0665b6efa86a1c3e947345731e4871ea3f9f64474fb1e3776d9ee9a4112696d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Linliu-Bioinfo/SSGATE.git
  tools:
  - SSGATE
  - anndata
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# triplet-loss-self-supervised-training

## Summary

Trains SSGATE's dual-path graph attention auto-encoder on paired transcriptome and proteome AnnData objects, combining per-modality mean-squared-error reconstruction losses with triplet-loss self-supervision into a weighted loss to produce a joint 30-dimensional cell embedding Z=[Temb|Pemb].

## When to use

Use when integrating paired transcriptome and proteome data (single-cell or spatial multi-omics) in AnnData format into a joint low-dimensional embedding via self-supervised dual-path graph attention auto-encoder training with triplet loss.

## When NOT to use

- Integrating omics modalities other than transcriptomics/proteomics — the paper notes that normalization and dimensionality reduction methods for other single-omics data are not yet available
- Data that has not been processed by Cal_Nbrs_Net and prune_net — the neighborhood network construction and pruning steps must precede training

## Inputs

- Transcriptome AnnData object with expression profiles in anndata.X and a pruned neighborhood graph in adata.uns['nbrs_net']
- Proteome AnnData object with expression profiles in anndata.X and a pruned neighborhood graph in adata.uns['nbrs_net']
- Spatial coordinates stored in anndata.obsm['spatial'] for spatially resolved data (optional; otherwise treated as single-cell)

## Outputs

- Joint integrated cell embedding of dimensionality 30 per cell (Z=[Temb|Pemb]), one row per input cell

## How to apply

Load the transcriptome and proteome AnnData objects, each first passed through Cal_Nbrs_Net (Radius or KNN model, feat='X_pca' or 'spatial') and prune_net so a pruned neighborhood graph resides in adata.uns['nbrs_net']. Build two GATE paths, each with a two-layer graph attention encoder (attention active in the first layer, inactive in the second), symmetric decoder, and hidden dimensionality of 128 per modality with an output embedding dimension of 30. In each epoch, encode both modalities and concatenate to form the joint representation Z=[Temb|Pemb]. Optimize the weighted loss L=(L_recon^t+L_recon^p)+0.1*(L_super^t+L_super^p), where reconstruction uses mean squared error and self-supervision uses triplet loss, with learning rate 0.001 for up to 300 epochs via back propagation. When the preset maximum epoch count is reached, stop training and export the joint representation (integrated embeddings).

## Related tools

- **SSGATE** (Provides the dual-path graph attention auto-encoder and the ssgate.train routine implementing triplet-loss self-supervision plus MSE reconstruction training) — https://github.com/Linliu-Bioinfo/SSGATE.git
- **anndata** (Required input container format: expression profiles must be stored in anndata.X and spatial coordinates in anndata.obsm['spatial'])

## Examples

```
adata_st, adata_sp = ssgate.train(adata_st, adata_sp, hidden_dims1=128, hidden_dims2=128, out_dims=30, epochs_init=50, n_epochs=300, lr=0.001, sigma=0.1)
```

## Evaluation signals

- Output embedding has dimensionality 30 per cell and row count matches the input cell count
- Training loss (weighted combination of MSE reconstruction and triplet self-supervision terms) converges over the 300 training epochs at learning rate 0.001
- adata.uns['nbrs_net'] exists for both modalities after Cal_Nbrs_Net and prune_net, with each edge connecting cells within the same cell population
- Joint representation is formed by concatenation Z=[Temb|Pemb] with each modality contributing a 30-dimensional block

## Limitations

- Hyperparameters are set 'based on experience' as defaults rather than systematically derived
- Integration of omics types beyond transcriptome/proteome lacks reasonable normalization and dimensionality reduction methods
- If spatial coordinates are not stored in anndata.obsm['spatial'], the data will be processed as single-cell multi-omics and spatial information cannot be used

## Evidence

- [abstract] It can also perform self-supervised learning for integration through the graph attention auto-encoders from two paths.: "It can also perform self-supervised learning for integration through the graph attention auto-encoders from two paths."
- [readme] After completing the above two steps, training can be carried out. After multiple rounds of training, the joint feature: "After completing the above two steps, training can be carried out. After multiple rounds of training, the joint feature representation of the two omics can be obtained."
- [readme] This function is used to prune the adjacency network calculated by the `Cal_Nbrs_Net` function, ensuring that each edge: "This function is used to prune the adjacency network calculated by the `Cal_Nbrs_Net` function, ensuring that each edge only connects cells within the same cell population."
- [discussion] Based on experience, we provide a set of hyperparameters suitable for most datasets as default settings: "Based on experience, we provide a set of hyperparameters suitable for most datasets as default settings"
