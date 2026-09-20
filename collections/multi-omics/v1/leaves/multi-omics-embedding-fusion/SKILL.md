---
name: multi-omics-embedding-fusion
description: Use when you have paired transcriptomic and proteomic single-cell or
  spatial multi-omics data in AnnData format with precomputed neighborhood graphs
  (adata.uns['nbrs_net']) and need a fused low-dimensional joint embedding per cell
  for downstream analysis.
when_to_use_negative:
- Input omics types are beyond transcriptome/proteome — the paper notes integration
  of other single-omics data still requires reasonable normalization and dimensionality
  reduction methods not yet available
- Neighborhood graphs have not been constructed via Cal_Nbrs_Net and prune_net — training
  requires adata.uns['nbrs_net'] per modality
- Data are not in AnnData format with expression profiles in anndata.X — the README
  states data cannot be read correctly otherwise
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SSGATE
  role: Dual-path graph attention auto-encoder that performs the self-supervised multi-omics
    integration; provides Cal_Nbrs_Net, prune_net, and the ssgate.train entry point
  repo: https://github.com/Linliu-Bioinf/SSGATE
- name: anndata
  role: Input data container format; expression profiles must be stored in anndata.X
    and spatial coordinates in anndata.obsm['spatial']
  repo: https://anndata.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:a96cffae9a782c10bb920d8a28b9a326ccc6de5c0f3bb7185ebbade09675c047
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Linliu-Bioinf/SSGATE
  tools:
  - SSGATE
  - anndata
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# multi-omics-embedding-fusion

## Summary

Fuses paired transcriptome and proteome AnnData objects into a single joint cell embedding (Z=[Temb|Pemb]) by training the SSGATE dual-path graph attention auto-encoder with combined MSE reconstruction and triplet self-supervision losses. Enables integrated multi-omics representation for both single-cell and spatially resolved data.

## When to use

Use when you have paired transcriptomic and proteomic single-cell or spatial multi-omics data in AnnData format with precomputed neighborhood graphs (adata.uns['nbrs_net']) and need a fused low-dimensional joint embedding per cell for downstream analysis.

## When NOT to use

- Input omics types are beyond transcriptome/proteome — the paper notes integration of other single-omics data still requires reasonable normalization and dimensionality reduction methods not yet available
- Neighborhood graphs have not been constructed via Cal_Nbrs_Net and prune_net — training requires adata.uns['nbrs_net'] per modality
- Data are not in AnnData format with expression profiles in anndata.X — the README states data cannot be read correctly otherwise

## Inputs

- Transcriptome AnnData object with expression matrix in anndata.X and pruned neighborhood graph in adata.uns['nbrs_net']
- Proteome AnnData object with expression matrix in anndata.X and pruned neighborhood graph in adata.uns['nbrs_net']
- Spatial coordinates in anndata.obsm['spatial'] (optional; absence causes single-cell mode)

## Outputs

- Joint integrated embedding per cell of dimensionality 30 (Z=[Temb|Pemb]) from the concatenation of the two GATE-path embeddings
- Trained dual-path GATE model parameters after up to 300 epochs
- Updated AnnData objects returned by ssgate.train (adata_st, adata_sp)

## How to apply

First ensure each modality is an AnnData object with expression in anndata.X (spatial coordinates in anndata.obsm['spatial'] if applicable) and a pruned neighborhood graph in adata.uns['nbrs_net'] produced by Cal_Nbrs_Net and prune_net. Build two GATE paths, each with a two-layer graph attention encoder (attention active in the first layer, deactivated in the second), a symmetric decoder with ReLU/Tanh activations, hidden_dims of 128 per modality, and out_dims of 30. In each training epoch, encode both modalities and concatenate their embeddings to form Z=[Temb|Pemb]. Train for up to 300 epochs (n_epochs=300, lr=0.001) minimizing the weighted loss L=(L_recon^t+L_recon^p)+0.1*(L_super^t+L_super^p), where reconstruction is mean squared error and self-supervision is triplet loss; parameters update by back propagation. Stop at the preset maximum epoch count and export the joint representation, verifying each cell's embedding is 30-dimensional and matches the input cell count.

## Related tools

- **SSGATE** (Dual-path graph attention auto-encoder that performs the self-supervised multi-omics integration; provides Cal_Nbrs_Net, prune_net, and the ssgate.train entry point) — https://github.com/Linliu-Bioinf/SSGATE
- **anndata** (Input data container format; expression profiles must be stored in anndata.X and spatial coordinates in anndata.obsm['spatial']) — https://anndata.readthedocs.io/en/latest/

## Examples

```
adata_st, adata_sp = ssgate.train(adata_st, adata_sp, hidden_dims1=128, hidden_dims2=128, out_dims=30, n_epochs=300, lr=0.001, sigma=0.1)
```

## Evaluation signals

- Output embedding dimensionality is 30 per cell (out_dims=30) and matches the input cell count
- Training loss converges over the 300 epochs under the combined loss L=(L_recon^t+L_recon^p)+0.1*(L_super^t+L_super^p)
- Both modalities' AnnData objects contain a pruned neighbor network stored in uns['nbrs_net'] before training begins
- Joint embedding facilitates downstream analysis with better performance and stronger robustness than competitive methods (TotalVI, scMM, MOFA+, etc.) on the paper's benchmark datasets

## Limitations

- Hyperparameters are set 'based on experience' as defaults rather than systematically derived
- Integration of omics beyond transcriptome/proteome lacks established normalization and dimensionality reduction methods
- No software versions are reported for compared methods; hyperparameter defaults like n_epochs appear inconsistent across README versions (200 vs 300)
- Optimization of the workflow, efficiency, and applicability to additional omics types is deferred to future work

## Evidence

- [full_text] SSGATE computes the self-supervision loss using a triplet loss function and the self-reconstruction loss using a mean squared error loss; these are combined into the weighted loss L: [evidence span withheld — anchor status: not_found]
- [readme] a two-layer graph attention encoder with attention activated in the first layer and deactivated in the second, hidden_dims 128, out_dims 30: "The dimensionality of the output layer. This is the size of the embedding space where the integrated features from both datasets will be projected."
- [abstract] It can also perform self-supervised learning for integration through the graph attention auto-encoders from two paths.: "It can also perform self-supervised learning for integration through the graph attention auto-encoders from two paths."
- [readme] SSGATE requires data in anndata format; expression profiles must be in anndata.X: "The expression profiles are required to be stored in anndata.X, otherwise the data cannot be read correctly."
- [readme] After multiple rounds of training, the joint feature representation of the two omics can be obtained.: "After multiple rounds of training, the joint feature representation of the two omics can be obtained."
