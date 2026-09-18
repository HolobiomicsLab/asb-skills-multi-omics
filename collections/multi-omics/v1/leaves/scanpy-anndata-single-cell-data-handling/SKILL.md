---
name: scanpy-anndata-single-cell-data-handling
description: Use when single-cell count data (e.g., scRNA-seq) must be loaded into
  a scanpy AnnData object with raw counts and responses stored in obsm before fitting
  VIVS or similar models, or when the gene set must be reduced to a manageable size.
when_to_use_negative:
- The AnnData already contains normalized expression rather than raw counts — VIVS
  only supports assays where X corresponds to gene expression counts
- The dataset has only a few thousand genes and no response variables in obsm to condition
  on — no filtering or obsm setup is needed
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scanpy
  role: Provides the AnnData object format used to hold raw counts and obsm responses
    for VIVS
- name: VIVS
  role: Consumes the prepared AnnData; its select_genes function performs the top-2000-gene
    filtering step
  repo: https://github.com/YosefLab/VIVS
- name: scVI
  role: Generative model whose likelihood operates on the raw counts stored in the
    AnnData
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:6f39406a4c85def178be4a6eed1135925d7e70e33b8935779fdb6c240baecd52
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 9
---

# scanpy-anndata-single-cell-data-handling

## Summary

Prepare single-cell multiomics data as a scanpy AnnData object with raw counts and response variables in `obsm`, then filter to top variable genes (e.g., n_top_genes=2000 via vivs select_genes) so downstream models like VIVS can consume them. This is the required data-format entry point for VI-VS feature-dependency analyses.

## When to use

Use when single-cell count data (e.g., scRNA-seq) must be loaded into a scanpy AnnData object with raw counts and responses stored in obsm before fitting VIVS or similar models, or when the gene set must be reduced to a manageable size.

## When NOT to use

- The AnnData already contains normalized expression rather than raw counts — VIVS only supports assays where X corresponds to gene expression counts
- The dataset has only a few thousand genes and no response variables in obsm to condition on — no filtering or obsm setup is needed

## Inputs

- Single-cell expression data (e.g., semi-synthetic PBMC counts) loadable as a scanpy AnnData object
- Raw gene expression counts in AnnData.X (not normalized data)
- Response variable(s) Y stored in AnnData.obsm as an array or dataframe

## Outputs

- A raw-count scanpy AnnData object filtered to the top 2000 genes via select_genes
- An AnnData whose obsm key (e.g., 'my_obsm_key') can be referenced as the VIVS feature_obsm_key

## How to apply

Import the data of interest into a scanpy AnnData object, ensuring the X matrix holds raw gene expression counts rather than normalized values, since VIVS only supports assays where X corresponds to gene expression counts. Store the response variable(s) Y of interest in the AnnData `obsm` attribute, either as an array or a dataframe, and reference them by key (e.g., feature_obsm_key="my_obsm_key"). Because VIVS may not scale to many thousands of genes, filter the object with `from vivs import select_genes; adata = select_genes(adata, n_top_genes=2000)`, keeping the top 2000 genes. The resulting filtered AnnData is then passed directly to VIVS for model training and hierarchical importance testing. Rationale: the generative model (e.g., scVI) requires count-scale data and the CRT importance computations are expensive enough that gene filtering is recommended by the authors.

## Related tools

- **scanpy** (Provides the AnnData object format used to hold raw counts and obsm responses for VIVS)
- **VIVS** (Consumes the prepared AnnData; its select_genes function performs the top-2000-gene filtering step) — https://github.com/YosefLab/VIVS
- **scVI** (Generative model whose likelihood operates on the raw counts stored in the AnnData)

## Examples

```
from vivs import select_genes; adata = select_genes(adata, n_top_genes=2000)
```

## Evaluation signals

- AnnData.X contains raw integer-scale counts, not library-size-normalized or log-transformed values
- The response matrix is accessible at the intended obsm key and its rows align one-to-one with AnnData cells
- After select_genes(adata, n_top_genes=2000), the number of retained genes is <= 2000
- The same obsm key passed to VIVS (feature_obsm_key) resolves without KeyError and the VIVS model initializes and trains via model.train_all()

## Limitations

- VIVS may not scale to many thousands of genes, so gene filtering is recommended before running
- VIVS only supports assays where X corresponds to gene expression counts; other assay types (e.g., chromatin accessibility) may require an adapted generative model
- No software version information is provided for the generative model or VI-VS implementation, which may affect reproducibility

## Evidence

- [readme] import the data of intest in a scanpy AnnData object.: "To use VIVS in your project, import the data of intest in a scanpy AnnData object."
- [readme] make sure than the anndata contains raw counts and not normalized data: "In particular, make sure than the anndata contains raw counts and not normalized data."
- [readme] The response(s) Y of interest are expected to be stored in the obsm attribute: "The response(s) $Y$ of interest are expected to be stored in the `obsm` attribute of the anndata object, either as an array or as a dataframe."
- [methods] adata = select_genes(adata, n_top_genes=2000): "adata = select_genes(adata, n_top_genes=2000)"
- [methods] import the data of intest in a scanpy AnnData object: "import the data of intest in a scanpy AnnData object."
