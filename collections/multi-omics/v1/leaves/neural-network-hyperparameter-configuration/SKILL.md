---
name: neural-network-hyperparameter-configuration
description: Use when training SSGATE's dual-path GATE on paired transcriptome/proteome
  AnnData objects and you need the paper's concrete defaults (hidden 128, out 30,
  300 epochs, lr 0.001, sigma 0.1).
when_to_use_negative:
- The neighborhood graphs have not yet been built or pruned (Cal_Nbrs_Net and prune_net
  not run) — training hyperparameters cannot be applied until this prerequisite is
  done
- Integrating omics modalities beyond transcriptome/proteome — the discussion notes
  these still require reasonable normalization and dimensionality reduction methods
  that are not yet available
- Inputs are not AnnData objects with expression in anndata.X — SSGATE cannot read
  them correctly, so tuning its training parameters is premature
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0121
tools:
- name: SSGATE
  role: Python package whose ssgate.train hyperparameters (hidden_dims1/2, out_dims,
    n_epochs, lr, sigma, cluster_update_epoch, epochs_init) this skill configures
  repo: https://github.com/Linliu-Bioinf/SSGATE.git
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:e03197bdffeb5cad72ee58ec8fcc3d5a865a3d0293f6aaf9d89c6adc76c85e1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Linliu-Bioinf/SSGATE
  tools:
  - SSGATE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# neural-network-hyperparameter-configuration

## Summary

Configures the training hyperparameters of SSGATE's dual-path graph attention auto-encoder (hidden_dims 128 per modality, out_dims 30, 300 epochs, lr 0.001, sigma 0.1) so paired transcriptome and proteome AnnData objects yield a converged joint embedding Z=[Temb|Pemb].

## When to use

Use when training SSGATE's dual-path GATE on paired transcriptome/proteome AnnData objects and you need the paper's concrete defaults (hidden 128, out 30, 300 epochs, lr 0.001, sigma 0.1).

## When NOT to use

- The neighborhood graphs have not yet been built or pruned (Cal_Nbrs_Net and prune_net not run) — training hyperparameters cannot be applied until this prerequisite is done
- Integrating omics modalities beyond transcriptome/proteome — the discussion notes these still require reasonable normalization and dimensionality reduction methods that are not yet available
- Inputs are not AnnData objects with expression in anndata.X — SSGATE cannot read them correctly, so tuning its training parameters is premature

## Inputs

- Transcriptome AnnData object with expression in anndata.X and pruned graph in uns['nbrs_net'] (from Cal_Nbrs_Net + prune_net)
- Proteome AnnData object with expression in anndata.X and pruned graph in uns['nbrs_net']

## Outputs

- Joint 30-dimensional embedding Z=[Temb|Pemb] with one row per input cell
- Trained dual-path GATE parameters plus the per-epoch training-loss trajectory over the 300 epochs

## How to apply

Before any training configuration, build and prune a neighborhood graph per modality with Cal_Nbrs_Net and prune_net so each AnnData carries its graph in uns['nbrs_net']. Set hidden_dims1=hidden_dims2=128 for the two graph attention encoder paths (ReLU/Tanh activations, attention activated in the first layer and deactivated in the second) and out_dims=30 for the embedding space. Train with n_epochs=300, lr=0.001, epochs_init=50, and cluster_update_epoch=20, minimizing L=(L_recon^t+L_recon^p)+0.1*(L_super^t+L_super^p), where sigma=0.1 scales the triplet-loss self-supervision terms against the MSE reconstruction terms. Stop when the preset epoch cap is reached and export the integrated embeddings; the paper reports these experience-based defaults as suitable for most datasets. Note the README documents an n_epochs default of 200, so pass 300 explicitly to match the paper's setting.

## Related tools

- **SSGATE** (Python package whose ssgate.train hyperparameters (hidden_dims1/2, out_dims, n_epochs, lr, sigma, cluster_update_epoch, epochs_init) this skill configures) — https://github.com/Linliu-Bioinf/SSGATE.git

## Examples

```
adata_st, adata_sp = ssgate.train(adata_st, adata_sp, hidden_dims1=128, hidden_dims2=128, out_dims=30, cluster_update_epoch=20, epochs_init=50, n_epochs=300, lr=0.001, sigma=0.1)
```

## Evaluation signals

- Output embedding has exactly 30 columns (out_dims=30) per cell and its row count matches the input cell count
- Training loss converges over the 300 epochs per the paper's reported hyperparameter settings
- Loss composition is correct: mean squared error for self-reconstruction and triplet loss for self-supervision, with the supervision terms weighted by 0.1 (sigma)
- Both GAT paths use hidden_dims of 128 with attention active only in the first layer and deactivated in the second
- Parameters are set explicitly rather than relying on README defaults — e.g., n_epochs must be 300 although the documented default is 200

## Limitations

- Hyperparameters are set 'based on experience' as defaults suitable for most datasets, not systematically derived or optimized per dataset
- README documents an n_epochs default of 200 while the paper's workflow trains for 300 epochs — defaults can drift, so pass values explicitly
- These settings apply only to the training stage; the workflow, efficiency, and applicability to additional omics types remain future work

## Evidence

- [readme] hidden_dims/out_dims defaults: "The dimensionality of the output layer. This is the size of the embedding space where the integrated features from both datasets will be projected."
- [discussion] experience-based defaults: "Based on experience, we provide a set of hyperparameters suitable for most datasets as default settings"
- [methods] loss functions used in training: [evidence span withheld — anchor status: not_found]
- [methods] architecture dimensions per modality: [evidence span withheld — anchor status: not_found]
- [methods] validation criteria: [evidence span withheld — anchor status: not_found]
