---
name: paired-autoencoder-reconstruction-optimization
description: Use when two paired, expression-matched AnnData objects (e.g., transcriptome
  and proteome, single-cell or spatial) each carrying a pruned neighborhood graph
  in adata.uns['nbrs_net'] must be fused into a single low-dimensional joint representation.
when_to_use_negative:
- Data are not paired across modalities (cells/spots cannot be matched between the
  two AnnData objects)
- Omics types beyond transcriptome/proteome that lack established normalization and
  dimensionality reduction methods for this workflow
- Neighborhood graphs have not yet been built and pruned via Cal_Nbrs_Net and prune_net
  — the training step requires adata.uns['nbrs_net']
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3940
- http://edamontology.org/topic_3170
tools:
- name: SSGATE
  role: Dual-path graph attention auto-encoder implementation providing Cal_Nbrs_Net,
    prune_net, and the ssgate.train routine that optimizes the combined reconstruction
    + triplet loss
  repo: https://github.com/Linliu-Bioinf/SSGATE
- name: anndata
  role: 'Input data container: expression must be in anndata.X and spatial coordinates
    in anndata.obsm[''spatial'']'
  repo: https://anndata.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:a6447810aaca7288725129203118db53f59be850229390c06574f2dc7e6f7b43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Linliu-Bioinf/SSGATE
  tools:
  - SSGATE
  - anndata
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3940
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# paired-autoencoder-reconstruction-optimization

## Summary

Jointly trains dual-path graph attention auto-encoders (SSGATE) on paired transcriptome and proteome AnnData objects, optimizing a weighted combination of MSE self-reconstruction loss and triplet self-supervision loss to produce a 30-dimensional joint embedding Z=[Temb|Pemb] per cell.

## When to use

Use when two paired, expression-matched AnnData objects (e.g., transcriptome and proteome, single-cell or spatial) each carrying a pruned neighborhood graph in adata.uns['nbrs_net'] must be fused into a single low-dimensional joint representation.

## When NOT to use

- Data are not paired across modalities (cells/spots cannot be matched between the two AnnData objects)
- Omics types beyond transcriptome/proteome that lack established normalization and dimensionality reduction methods for this workflow
- Neighborhood graphs have not yet been built and pruned via Cal_Nbrs_Net and prune_net — the training step requires adata.uns['nbrs_net']

## Inputs

- Transcriptome AnnData object with expression in adata.X and pruned neighbor graph in adata.uns['nbrs_net']
- Proteome AnnData object with expression in adata.X and pruned neighbor graph in adata.uns['nbrs_net']
- Spatial coordinates in adata.obsm['spatial'] for spatially resolved data (optional)

## Outputs

- Integrated 30-dimensional joint embedding Z=[Temb|Pemb] per cell
- Trained dual-path GATE model parameters
- Training loss trajectory over epochs

## How to apply

First ensure each modality has been processed by Cal_Nbrs_Net (feat='X_pca' or 'spatial', model='Radius' with rad_cutoff or 'KNN' with k_cutoff) and prune_net, storing the pruned graph in adata.uns['nbrs_net']. Then call ssgate.train on both AnnData objects with hidden_dims1=hidden_dims2=128 and out_dims=30; each path is a two-layer graph attention encoder (attention active in the first layer, deactivated in the second) with a symmetric decoder and ReLU/Tanh activations. Per epoch, encode both modalities and form the joint representation by concatenation Z=[Temb|Pemb]. Optimize the combined weighted loss L=(L_recon^t+L_recon^p)+0.1*(L_super^t+L_super^p), where reconstruction uses mean squared error and self-supervision uses triplet loss, via backpropagation with lr=0.001 for up to 300 epochs (cluster_update_epoch=20, epochs_init=50, sigma=0.1 per the README defaults). Stop at the preset maximum epoch count and export the integrated embeddings.

## Related tools

- **SSGATE** (Dual-path graph attention auto-encoder implementation providing Cal_Nbrs_Net, prune_net, and the ssgate.train routine that optimizes the combined reconstruction + triplet loss) — https://github.com/Linliu-Bioinf/SSGATE
- **anndata** (Input data container: expression must be in anndata.X and spatial coordinates in anndata.obsm['spatial']) — https://anndata.readthedocs.io/en/latest/

## Examples

```
adata_st, adata_sp = ssgate.train(adata_st, adata_sp, hidden_dims1=128, hidden_dims2=128, out_dims=30, epochs_init=50, n_epochs=300, lr=0.001, sigma=0.1)
```

## Evaluation signals

- Output embedding has dimensionality 30 per cell (out_dims=30) and matches the input cell count
- Training loss converges over the 300 epochs under the reported hyperparameters (lr=0.001, hidden_dims=128)
- Spatially resolved inputs are handled with spatial information only if adata.obsm['spatial'] is populated; otherwise the data are treated as single-cell — verify this flag was read
- Downstream analysis (e.g., clustering on Z) performs comparably or better than competitive methods, per the paper's robustness claims

## Limitations

- Hyperparameters are set 'based on experience' as defaults rather than systematically derived
- Normalization and dimensionality reduction methods for integrating omics beyond transcriptome/proteome are not yet available
- README defaults show n_epochs=200 in one listing while the paper-reported training uses up to 300 epochs; the epoch count is a decision point
- Expression profiles must be stored in anndata.X and spatial coordinates in anndata.obsm['spatial'], or data cannot be read correctly

## Evidence

- [results] During training, SSGATE computes the self-supervision loss using a triplet loss function and the self-reconstruction loss using a mean squared error loss: [evidence span withheld — anchor status: not_found]
- [results] the joint representation by concatenation Z=[Temb|Pemb]: [evidence span withheld — anchor status: not_found]
- [readme] SSGATE requires data in anndata format as inputs. The expression profiles are required to be stored in anndata.X: "SSGATE requires data in [anndata](https://anndata.readthedocs.io/en/latest/) format as inputs. The expression profiles are required to be stored in anndata.X, otherwise the data cannot be read"
- [readme] hidden_dims1 = 128, hidden_dims2 = 128, out_dims = 30, ... lr=0.001: "hidden_dims1 = 128, 
                                      hidden_dims2 = 128, 
                                      out_dims = 30, 
                                      cluster_update_epoch = 20,"
- [abstract] It can also perform self-supervised learning for integration through the graph attention auto-encoders from two paths.: "It can also perform self-supervised learning for integration through the graph attention auto-encoders from two paths."
