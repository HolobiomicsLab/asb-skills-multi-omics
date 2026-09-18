---
name: conditional-randomization-test-implementation
description: Use when testing conditional independence between gene expression features
  and a response (protein counts, synthetic y, perturbation readouts) in single-cell
  data with a requirement for calibrated FDR control without linearity or Gaussian-response
  assumptions.
when_to_use_negative:
- The AnnData contains normalized rather than raw counts — VIVS only supports assays
  where X corresponds to raw gene expression counts
- The dataset has many thousands of genes without prior filtering — VIVS may not scale;
  use select_genes(adata, n_top_genes=2000) first
- The goal is chromatin accessibility (ATAC-seq) — the paper notes this may require
  a generative model that properly accounts for ATAC-seq sparsity to keep p-values
  valid
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: VIVS
  role: 'Python package implementing the CRT pipeline: scVI-style generative model
    training, synthetic sampling from the decoder, and p-value computation'
  repo: github.com/YosefLab/VIVS
- name: Jax
  role: Backend for parallel computing and just-in-time compilation to speed up inference;
    install with GPU support
- name: scVI
  role: Latent-variable VAE generative model of gene counts used to draw synthetic
    feature replicates conditioned on latent z and nuisance factors s
- name: scanpy
  role: 'Data container: import raw-count AnnData and responses from obsm before running
    VIVS'
- name: GENIE3
  role: Example external model whose regression-tree prediction errors can be plugged
    in as importance scores T_phi for calibrated discoveries
merged_aliases:
- conditional-randomization-testing
merged_alias_records:
- alias: conditional-randomization-testing
  slug: conditional-randomization-testing
  cosine_score: 0.8952
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_008
  - task_006
  - task_007
  - task_004
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:8a011e5789f14d8671bd37a647fc69bd4bace26c721b48be963815c72cd7c9ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VIVS
  tools:
  - GENIE3
  - Jax
  - PyTorch
  - Python
  - VI-VS
  - VIVS
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# conditional-randomization-test-implementation

## Summary

Implements the VI-VS conditional randomization test (CRT): a scVI-style VAE generative model of gene counts (conditioned on nuisance factors S, never seeing the response y) generates K synthetic replicates of each gene from a fixed posterior sample z-bar, and importance scores T(X, Y, S) computed on held-out data yield calibrated per-gene CRT p-values for conditional independence testing.

## When to use

Use when testing conditional independence between gene expression features and a response (protein counts, synthetic y, perturbation readouts) in single-cell data with a requirement for calibrated FDR control without linearity or Gaussian-response assumptions.

## When NOT to use

- The AnnData contains normalized rather than raw counts — VIVS only supports assays where X corresponds to raw gene expression counts
- The dataset has many thousands of genes without prior filtering — VIVS may not scale; use select_genes(adata, n_top_genes=2000) first
- The goal is chromatin accessibility (ATAC-seq) — the paper notes this may require a generative model that properly accounts for ATAC-seq sparsity to keep p-values valid

## Inputs

- scanpy AnnData object with raw gene expression counts (not normalized)
- Response variable(s) Y stored in the obsm attribute (array or dataframe, e.g., CITE-seq protein counts or synthetic responses)
- Nuisance factors S (e.g., batch, sequencing depth, cell cycle)
- Number of Monte Carlo replicates K per feature

## Outputs

- Per-feature CRT p-values p_g = (1 + sum_k indicator)/(K+1)
- BH-adjusted discovery calls at target FDR levels (e.g., alpha=0.1)
- Summary tables of empirical FDR and TPR per method and target level (e.g., fdr_tpr_table.csv)

## How to apply

Load data as a scanpy AnnData object with raw counts and the response Y in obsm, filter genes with select_genes(adata, n_top_genes=2000) since VIVS may not scale to many thousands of genes, and split cells into development and held-out validation sets. Fit the VIVS model (e.g., xy_linear=False, xy_model_kwargs={'n_hidden': 8} for an MLP importance score) on the development split so the generative model p(x, z | s) = p(z) * prod_g p(x_g | z, s) never sees y. For each tested gene g, draw one fixed posterior sample z-bar ~ q(z | x^n) per validation cell, generate K i.i.d. synthetic vectors from the decoder p_theta(x_g | z-bar, s) replacing only gene g, and compute T(X, Y, S) = (1/N) sum_n -log p_phi(y^n | x^n, s^n) for observed and synthetic replicates. Compute the CRT p-value p_g = (1 + sum_k 1{T(X-tilde^(k), Y, S) <= T(X, Y, S)}) / (K + 1) and apply the Benjamini-Hochberg procedure at target FDR levels (e.g., 0.05, 0.1, 0.2). Calibrated p-values stochastically dominate the uniform distribution under the null, so empirical FDR should track but not exceed the nominal target.

## Related tools

- **VIVS** (Python package implementing the CRT pipeline: scVI-style generative model training, synthetic sampling from the decoder, and p-value computation) — github.com/YosefLab/VIVS
- **Jax** (Backend for parallel computing and just-in-time compilation to speed up inference; install with GPU support)
- **scVI** (Latent-variable VAE generative model of gene counts used to draw synthetic feature replicates conditioned on latent z and nuisance factors s)
- **scanpy** (Data container: import raw-count AnnData and responses from obsm before running VIVS)
- **GENIE3** (Example external model whose regression-tree prediction errors can be plugged in as importance scores T_phi for calibrated discoveries)

## Examples

```
model = VIVS(adata, feature_obsm_key="my_obsm_key", xy_linear=False, xy_model_kwargs={"n_hidden": 8}); model.train_all()
```

## Evaluation signals

- Under a null simulation where x_g is conditionally independent of y, the empirical distribution of p_g stochastically dominates the uniform distribution (valid p-value per Proposition 1)
- After Benjamini-Hochberg correction, realized FDR is at or below the nominal target level across replicates
- VI-VS empirical FDR stays calibrated in semi-synthetic PBMC experiments where the OLS t-test overestimates FDR and the marginal test exceeds target levels
- Importance-score models with lower held-out MSE yield higher TPR (power), consistent with the paper's expectation that held-out predictive performance guides score selection
- Output tables contain rows for all methods and target FDR levels with FDR and TPR columns

## Limitations

- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries
- VIVS may not scale to many thousands of genes, so genes must be pre-filtered (e.g., n_top_genes=2000)
- Only supports assays where X corresponds to gene expression counts; chromatin accessibility may require a sparsity-aware generative model for valid p-values
- No software version information is provided for the generative model or VI-VS implementation

## Evidence

- [results] the generative model and the importance scores are both trained on the development split of the data, while p-values (and cell scores) are computed on the held-out validation split: [evidence span withheld — anchor status: not_found]
- [methods] Compute the CRT p-value p_g = (1 + sum_k 1{T(X-tilde^(k), Y, S) <= T(X, Y, S)}) / (K + 1): [evidence span withheld — anchor status: not_found]
- [methods] verify that the empirical distribution of p_g stochastically dominates the uniform distribution (valid p-value per Proposition 1): [evidence span withheld — anchor status: not_found]
- [readme] In particular, make sure than the anndata contains raw counts and not normalized data.: "In particular, make sure than the anndata contains raw counts and not normalized data."
- [discussion] chromatin accessibility data may require the use of a generative model that property account for ATAC-seq sparsity [67, 68] to ensure that VI-VS p-values remain valid.: "chromatin accessibility data may require the use of a generative model that property account for ATAC-seq sparsity [67, 68] to ensure that VI-VS p-values remain valid."
