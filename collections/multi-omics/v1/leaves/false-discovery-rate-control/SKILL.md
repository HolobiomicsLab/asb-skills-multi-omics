---
name: false-discovery-rate-control
description: Use when single-cell count data (raw counts in a scanpy AnnData) must
  be screened for conditionally dependent features (e.g., gene-protein or gene-response
  links) while provably controlling the false discovery rate at a target level such
  as 0.05, 0.1, or 0.2.
when_to_use_negative:
- The anndata contains normalized rather than raw counts — VIVS only supports assays
  where X corresponds to gene expression counts
- The dataset has many thousands of genes without prior filtering — VIVS may not scale;
  select_genes(n_top_genes=2000) is recommended first
- The goal is marginal association or a purely heuristic relevance ranking without
  FDR guarantees — VI-VS is designed for calibrated conditional-dependency testing,
  not marginal co-expression networks
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: VI-VS / VIVS
  role: Implements the conditional randomization test with a deep generative model
    and provides select_genes, VIVS model training, and hierarchical importance calls;
    installed via 'pip install vivs' on Python >= 3.9
  repo: github.com/YosefLab/VIVS
- name: Jax
  role: Backend for parallel computing and just-in-time compilation to speed up CRT
    inference; install with GPU support
  repo: https://jax.readthedocs.io/en/latest/installation.html
- name: scVI
  role: Generative model for normalized gene expression (count data) used as the feature-sampling
    decoder in the CRT, e.g., conditioning on batch as a nuisance factor
- name: scanpy
  role: Data container format — inputs must be imported as a scanpy AnnData object
    with raw counts and responses in obsm
- name: GENIE3
  role: Existing pairwise-relationship algorithm whose discoveries can be wrapped
    by VI-VS as an importance score to yield calibrated decisions
merged_aliases:
- false-discovery-rate-estimation
- false-discovery-rate-multiple-testing-correction
merged_alias_records:
- alias: false-discovery-rate-estimation
  slug: false-discovery-rate-estimation
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
- alias: false-discovery-rate-multiple-testing-correction
  slug: false-discovery-rate-multiple-testing-correction
  cosine_score: 0.813
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_003
  - task_001
  - task_007
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:e956adf699c7add4f71ea4d07f7d1ebe34f009b962e7bd85af178b2d90e21781
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VIVS
  tools:
  - GENIE3
  - Jax
  - Python
  - VI-VS
  - VI-VS / VIVS
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 15
---

# false-discovery-rate-control

## Summary

Applies the VI-VS conditional randomization test (CRT) with a VAE/scVI-style generative model to compute per-feature p-values for conditional dependence in single-cell multiomics data, then calibrates decisions via the Benjamini-Hochberg procedure so empirical FDR stays at or below the target level.

## When to use

Use when single-cell count data (raw counts in a scanpy AnnData) must be screened for conditionally dependent features (e.g., gene-protein or gene-response links) while provably controlling the false discovery rate at a target level such as 0.05, 0.1, or 0.2.

## When NOT to use

- The anndata contains normalized rather than raw counts — VIVS only supports assays where X corresponds to gene expression counts
- The dataset has many thousands of genes without prior filtering — VIVS may not scale; select_genes(n_top_genes=2000) is recommended first
- The goal is marginal association or a purely heuristic relevance ranking without FDR guarantees — VI-VS is designed for calibrated conditional-dependency testing, not marginal co-expression networks

## Inputs

- scanpy AnnData object with raw gene expression counts (X) and response variables (e.g., proteins, synthetic y) stored in the obsm attribute
- Target FDR levels (e.g., 0.05, 0.1, 0.2)
- Number K of Monte Carlo samples per gene for the CRT
- Optional nuisance covariates (e.g., 13 batch assignments) for the generative model

## Outputs

- Per-feature (conditional randomization test) p-values
- Benjamini-Hochberg-adjusted selections at each target FDR level
- Empirical FDR and TPR summary table (e.g., fdr_tpr_table.csv or fig3bc_fdr_tpr_table.csv with importance_score, target_FDR, empirical_FDR, TPR, held-out MSE columns)
- Association tables (e.g., cite_seq_pbmc_associations.csv) listing detected gene-response pairs

## How to apply

Load the data as a scanpy AnnData object containing raw counts (not normalized data), with response variables Y stored in the obsm attribute, and filter to top genes with select_genes(adata, n_top_genes=2000) since VIVS may not scale to many thousands of genes. Initialize and train VIVS (model.train_all()) so the deep generative model captures feature dependencies independently of the response, choosing the importance-score model via xy_linear=False with xy_model_kwargs={'n_hidden': 8} for nonlinear (MLP) scores, and conditioning on nuisance factors such as batch assignments in the generative model. For each feature, run the conditional randomization test: draw K Monte Carlo synthetic samples of the feature from the VAE decoder (holding Y and S fixed) and compare the statistic T(X_tilde, Y, S) to T(X, Y, S), computing p-values as pg = (1 + number of indicator terms)/(K+1). Apply the Benjamini-Hochberg procedure to the per-feature p-values at the target FDR levels (e.g., 0.05, 0.1, 0.2) to obtain calibrated selections. The choice of importance score matters for power, not validity: lower held-out MSE predictors yield higher TPR while FDR remains controlled, so compare held-out MSE when selecting between linear OLS and MLP scores.

## Related tools

- **VI-VS / VIVS** (Implements the conditional randomization test with a deep generative model and provides select_genes, VIVS model training, and hierarchical importance calls; installed via 'pip install vivs' on Python >= 3.9) — github.com/YosefLab/VIVS
- **Jax** (Backend for parallel computing and just-in-time compilation to speed up CRT inference; install with GPU support) — https://jax.readthedocs.io/en/latest/installation.html
- **scVI** (Generative model for normalized gene expression (count data) used as the feature-sampling decoder in the CRT, e.g., conditioning on batch as a nuisance factor)
- **scanpy** (Data container format — inputs must be imported as a scanpy AnnData object with raw counts and responses in obsm)
- **GENIE3** (Existing pairwise-relationship algorithm whose discoveries can be wrapped by VI-VS as an importance score to yield calibrated decisions)

## Examples

```
from vivs import VIVS; model = VIVS(adata, feature_obsm_key="my_obsm_key", xy_linear=False, xy_model_kwargs={"n_hidden": 8}); model.train_all(); res = model.get_hier_importance(n_clusters_list=[100, 200])
```

## Evaluation signals

- Empirical FDR (false discoveries / total calls) at or below each nominal target level (e.g., 0.05, 0.1, 0.2) across replicates
- Summary table exists with all expected columns (importance_score, target_FDR, empirical_FDR, TPR, held-out MSE) and rows for all methods and target levels
- More adequate importance-score predictors (lower held-out MSE) yield higher TPR while FDR stays calibrated, consistent with the paper's expectation
- Negative-control features (e.g., Rat-IgG isotype proteins in CITE-seq) show detections consistent with the target FDR rather than mass discovery
- CRT p-values computed with the (1 + sum of indicator terms)/(K+1) convention and processed through Benjamini-Hochberg

## Limitations

- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries
- Marginal association methods inflate correlations due to technical factors (batch effects, sequencing depth) and nuisance biological factors (e.g., cell cycle), which is why conditional testing with nuisance correction is required
- Applicability to chromatin accessibility data is not yet demonstrated and may require a generative model that properly accounts for ATAC-seq sparsity to keep p-values valid
- OLS t-test baselines overestimated the FDR in the semi-synthetic PBMC experiment due to violated linearity/Gaussian assumptions; the marginal independence test exceeded target FDR levels

## Evidence

- [abstract] It uses deep generative models to identify conditionally dependent features, with false discovery rate control.: "It uses deep generative models to identify conditionally dependent features, with false discovery rate control."
- [intro] We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate.: "We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate."
- [methods] import the data of intest in a scanpy AnnData object.: "import the data of intest in a scanpy AnnData object."
- [readme] Right now, VIVS only supports assays where $X$ corresponds to gene expression counts. In particular, make sure than the anndata contains raw counts and not normalized data.: "Right now, VIVS only supports assays where $X$ corresponds to gene expression counts. In particular, make sure than the anndata contains raw counts"
- [readme] In such a case, it is recommended to filter the genes before running VIVS: "VIVS may not scale to many thousands of genes. In such a case, it is recommended to filter the genes before running VIVS"
