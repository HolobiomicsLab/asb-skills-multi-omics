---
name: log-cpm-feature-engineering
description: Use when raw gene expression counts in a scanpy AnnData object must be
  converted to predictor features (e.g., squared log-CPM) for VI-VS variable selection
  or conditional randomization testing.
when_to_use_negative:
- The AnnData X matrix contains normalized (log-normalized or scVI-denoised) values
  rather than raw counts — VIVS only supports gene expression counts for X
- The full gene set is many thousands of genes and no pre-filtering step is applied
  — VIVS may not scale to many thousands of genes
- The assay is not gene expression (e.g., chromatin accessibility), where generative-model
  sparsity assumptions for ATAC-seq have not yet been validated
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scanpy
  role: Loads and holds the raw-count AnnData object from which log-CPM features are
    computed
- name: VIVS
  role: Consumes the transformed features via feature_obsm_key for conditional dependency
    testing
  repo: github.com/YosefLab/VIVS
- name: Python
  role: Implementation environment (Python >= 3.9) for the transformation and downstream
    VIVS run
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:76569e7081abdb2a0c15e1e49d121c6deae9ad0a1d6a49a2bbf3793e5e39f760
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jax
  - Python
  - VI-VS
  - VIVS
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# log-cpm-feature-engineering

## Summary

Transform raw single-cell gene expression counts (scanpy AnnData) into squared log-CPM features to serve as predictor variables X for VI-VS conditional dependency testing, as in the semi-synthetic PBMC experiment where a subset of genes is transformed and Gaussian-noise responses are regressed on them.

## When to use

Use when raw gene expression counts in a scanpy AnnData object must be converted to predictor features (e.g., squared log-CPM) for VI-VS variable selection or conditional randomization testing.

## When NOT to use

- The AnnData X matrix contains normalized (log-normalized or scVI-denoised) values rather than raw counts — VIVS only supports gene expression counts for X
- The full gene set is many thousands of genes and no pre-filtering step is applied — VIVS may not scale to many thousands of genes
- The assay is not gene expression (e.g., chromatin accessibility), where generative-model sparsity assumptions for ATAC-seq have not yet been validated

## Inputs

- Raw gene expression counts in a scanpy AnnData object (e.g., PBMC 6,855-cell dataset)
- Number of top genes or a random gene subset (e.g., 150 random genes, or n_top_genes=2000 via select_genes)
- Total counts per cell used for the CPM normalization

## Outputs

- Squared log-CPM transformed predictor matrix (cells x selected genes), suitable for VIVS feature_obsm_key
- Gene list / indices of the selected or sampled predictor genes
- Feature matrix stored in the AnnData obsm attribute for downstream VIVS model initialization

## How to apply

Load the dataset of interest (e.g., the 6,855-cell PBMC dataset) as a scanpy AnnData object containing raw gene expression counts — VIVS requires raw counts, not normalized data. If the gene set is large, filter first with the vivs select_genes helper (e.g., n_top_genes=2000), since VIVS may not scale to many thousands of genes; the PBMC semi-synthetic workflow samples a subset of genes (150 random genes) as predictors. Compute log-CPM (counts per million) for the selected genes, take the log transform, then square the resulting values so the transformed features serve as the predictor matrix passed to VIVS via feature_obsm_key. Generate the response(s) Y separately (e.g., five synthetic responses from selected genes plus Gaussian noise) and store them in the AnnData obsm attribute, then fit VIVS with a nonlinear MLP importance score (xy_linear=False, xy_model_kwargs={'n_hidden': 8}) and run the conditional randomization test with Benjamini-Hochberg FDR control. The squared log-CPM transformation provides a nonlinear feature representation of expression magnitude so that the MLP importance score can capture conditional dependencies without linearity assumptions.

## Related tools

- **scanpy** (Loads and holds the raw-count AnnData object from which log-CPM features are computed)
- **VIVS** (Consumes the transformed features via feature_obsm_key for conditional dependency testing) — github.com/YosefLab/VIVS
- **Python** (Implementation environment (Python >= 3.9) for the transformation and downstream VIVS run)

## Examples

```
from vivs import select_genes; adata = select_genes(adata, n_top_genes=2000)
```

## Evaluation signals

- Verify the AnnData X contains raw counts, not normalized data, before computing CPM
- Verify the gene filter/subset step ran (e.g., select_genes with n_top_genes=2000 or a 150-gene random sample) and the feature matrix has shape (n_cells x n_selected_genes)
- Verify the transformed features are squared log-CPM: non-negative values, zero for cells with zero counts of the gene
- Verify responses Y are stored in the AnnData obsm attribute as an array or dataframe, matching the VIVS data-format requirement
- Downstream check: the VIVS model initializes with feature_obsm_key pointing at the transformed matrix and trains without shape or dtype errors

## Limitations

- VIVS may not scale to many thousands of genes, so genes must be filtered before feature construction
- VIVS only supports assays where X corresponds to gene expression counts; the response variables must be placed in the obsm attribute
- No software version information is provided for the VIVS implementation in the article

## Evidence

- [readme] To use VIVS in your project, import the data of intest in a scanpy AnnData object.: "To use VIVS in your project, import the data of intest in a scanpy AnnData object."
- [readme] VIVS only supports gene expression counts in X; raw counts required, not normalized data.: "Right now, VIVS only supports assays where $X$ corresponds to gene expression counts. In particular, make sure than the anndata contains raw counts and not normalized data."
- [readme] Filter genes before running VIVS with select_genes (n_top_genes=2000).: "adata = select_genes(adata, n_top_genes=2000)"
- [readme] Responses Y are expected in the obsm attribute of the AnnData object.: "The response(s) $Y$ of interest are expected to be stored in the `obsm` attribute of the anndata object, either as an array or as a dataframe."
- [methods] Import data as a scanpy AnnData object with raw counts (methods).: "import the data of intest in a scanpy AnnData object."
