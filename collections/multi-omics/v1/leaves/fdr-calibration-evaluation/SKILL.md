---
name: fdr-calibration-evaluation
description: Use when benchmarking FDR control of a variable-selection method on count-valued
  (Poisson-generated) semi-synthetic responses, where OLS-based importance scores
  are suspected to miscalibrate but CRT-based scores should stay valid.
when_to_use_negative:
- The data are chromatin accessibility (ATAC-seq) counts, since VI-VS p-values may
  require a generative model that properly accounts for ATAC-seq sparsity to remain
  valid
- The feature matrix is not raw gene expression counts in an AnnData object's X slot
  (VIVS only supports raw counts, not normalized data)
- The response is a Gaussian-noised linear signal where the simple OLS baseline is
  expected to be adequate, so the Poisson-count stress test is unnecessary
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: VIVS
  role: Provides the CRT-based p-values whose FDR calibration is evaluated; trains
    the VAE generative model on the count matrix
  repo: https://github.com/YosefLab/VIVS
- name: Jax
  role: Backend for parallel computing and just-in-time compilation to speed up VI-VS
    inference during the K-copy Monte Carlo CRT
- name: scVI
  role: Generative model for normalized gene expression underlying the conditional
    randomization test
- name: scanpy
  role: 'Data container: the semi-synthetic count matrix is loaded as a scanpy AnnData
    object before running VIVS'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:f349ac7505fffae4ac953d92fc877a6fb5b0ec7d60d93fa46a85a2644f717602
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VIVS
  tools:
  - Jax
  - Python
  - VI-VS
  - VIVS
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# fdr-calibration-evaluation

## Summary

Evaluates whether a feature-selection procedure (e.g., VI-VS vs. OLS) maintains false discovery rate calibration when semi-synthetic PBMC responses are generated as Poisson counts rather than Gaussian-noised linear responses. Realized FDR is compared against Benjamini-Hochberg target levels on conditional randomization test (CRT) p-values.

## When to use

Use when benchmarking FDR control of a variable-selection method on count-valued (Poisson-generated) semi-synthetic responses, where OLS-based importance scores are suspected to miscalibrate but CRT-based scores should stay valid.

## When NOT to use

- The data are chromatin accessibility (ATAC-seq) counts, since VI-VS p-values may require a generative model that properly accounts for ATAC-seq sparsity to remain valid
- The feature matrix is not raw gene expression counts in an AnnData object's X slot (VIVS only supports raw counts, not normalized data)
- The response is a Gaussian-noised linear signal where the simple OLS baseline is expected to be adequate, so the Poisson-count stress test is unnecessary

## Inputs

- Semi-synthetic PBMC gene expression count matrix with correlated features (raw counts in a scanpy AnnData object)
- Response variable y generated as Poisson counts conditionally dependent on a designated feature
- Nuisance/technical factor covariates used in the OLS regression
- Number K of randomized copies per gene for the Monte Carlo CRT

## Outputs

- CRT p-values per feature for VI-VS and for the OLS baseline
- Comparison table of realized FDR at each Benjamini-Hochberg target FDR level
- Plot of realized FDR versus target FDR level for both methods (mirroring Additional file 1: Figure S1)

## How to apply

Simulate a semi-synthetic feature matrix of correlated gene expression counts and draw the response y as Poisson counts conditionally dependent on a designated feature, replacing the Gaussian-noised linear response. Fit the VI-VS pipeline: train the VAE latent-variable generative model on the count matrix (Jax backend) and compute CRT p-values per feature using a Monte Carlo scheme with K randomized copies of each gene drawn from a fixed posterior sample z-bar. Fit the OLS baseline by computing importance scores as prediction error of a linear regression of y on X and nuisance factors, with CRT p-values from the same K randomizations. Apply the Benjamini-Hochberg procedure to both methods' p-values over a grid of target FDR levels, record realized FDR at each level, and plot realized versus target FDR curves for VI-VS and OLS.

## Related tools

- **VIVS** (Provides the CRT-based p-values whose FDR calibration is evaluated; trains the VAE generative model on the count matrix) — https://github.com/YosefLab/VIVS
- **Jax** (Backend for parallel computing and just-in-time compilation to speed up VI-VS inference during the K-copy Monte Carlo CRT)
- **scVI** (Generative model for normalized gene expression underlying the conditional randomization test)
- **scanpy** (Data container: the semi-synthetic count matrix is loaded as a scanpy AnnData object before running VIVS)

## Examples

```
from vivs import VIVS; model = VIVS(adata, feature_obsm_key="my_obsm_key", xy_linear=False, xy_model_kwargs={"n_hidden": 8}); model.train_all(); res = model.get_hier_importance(n_clusters_list=[100, 200])
```

## Evaluation signals

- A method is considered calibrated if its p-values stochastically dominate the uniform distribution under the null
- Realized FDR curve tracks the target FDR level across the grid of Benjamini-Hochberg target levels
- VI-VS with an MLP importance score controls the FDR in both simulation scenarios, while OLS should visibly exceed target FDR under Poisson-generated responses
- CRT p-values for both methods are computed with the same K randomized copies, so realized-FDR differences reflect the importance-score scheme, not the randomization count

## Limitations

- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries
- VIVS may not scale to many thousands of genes; genes should be filtered (e.g., select_genes with n_top_genes=2000) before running
- No software version information is provided for the generative model or VI-VS implementation, complicating exact reproduction

## Evidence

- [intro] CRT p-value calibration criterion: [evidence span withheld — anchor status: not_found]
- [methods] Jax backend for the generative-model inference: "Our implementation first relies on parallel computing and just-in-time compilation components of Jax to speed up the inference"
- [methods] Generative model used for normalized gene expression: "the normalized gene expression of the generative model, e.g., scVI [32]."
- [readme] Raw-counts requirement in AnnData: "Right now, VIVS only supports assays where $X$ corresponds to gene expression counts."
- [readme] Gene filtering before fitting, relevant to scalability of the calibration experiment: "VIVS may not scale to many thousands of genes.
In such a case, it is recommended to filter the genes before running VIVS"
