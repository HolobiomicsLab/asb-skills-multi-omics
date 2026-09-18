---
name: single-cell-multiomics-data-preprocessing
description: Use when input is paired transcriptome/proteome data in h5ad (anndata.X
  expression; spatial coordinates in anndata.obsm['spatial']) that must be normalized
  and dimension-reduced before neighborhood graph construction or multi-omics integration.
when_to_use_negative:
- Input is already a normalized feature/embedding table rather than raw per-cell counts
  requiring scaling, HVG selection, and PCA
- The omics modality is neither transcriptomic nor proteomic — the article notes normalization
  and dimensionality reduction for other single-omics data 'requires reasonable normalization
  and dimensionality reduction methods' not yet available
- Expression data is not stored in anndata.X, in which case SSGATE cannot read the
  data correctly
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SCANPY
  role: Performs count-depth scaling with log1p transformation, highly variable gene
    selection, PCA to 200 dimensions, and CLR transformation of proteomics data
- name: anndata
  role: 'Required input format: expression profiles must be in anndata.X and spatial
    coordinates in anndata.obsm[''spatial'']'
  repo: https://anndata.readthedocs.io/en/latest/
- name: SSGATE
  role: Downstream consumer of the preprocessed AnnData objects; PCA-reduced 'X_pca'
    features feed Cal_Nbrs_Net
  repo: https://github.com/Linliu-Bioinf/SSGATE
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:09edd0726ee78f9aae9e1981a9222f455ebf362690838f6fc9238932ca204e02
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MOFA
  - SCANPY
  - SSGATE
  - anndata
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# single-cell-multiomics-data-preprocessing

## Summary

Preprocess paired transcriptomic and proteomic single-cell/spatial multi-omics data into the AnnData format required by SSGATE: count-depth scaling with log1p plus highly variable gene selection and PCA (200 components) for transcriptomics, and centered log-ratio transformation for proteomics, using SCANPY.

## When to use

Use when input is paired transcriptome/proteome data in h5ad (anndata.X expression; spatial coordinates in anndata.obsm['spatial']) that must be normalized and dimension-reduced before neighborhood graph construction or multi-omics integration.

## When NOT to use

- Input is already a normalized feature/embedding table rather than raw per-cell counts requiring scaling, HVG selection, and PCA
- The omics modality is neither transcriptomic nor proteomic — the article notes normalization and dimensionality reduction for other single-omics data 'requires reasonable normalization and dimensionality reduction methods' not yet available
- Expression data is not stored in anndata.X, in which case SSGATE cannot read the data correctly

## Inputs

- Single-cell or spatially resolved multi-omics dataset in h5ad/AnnData format (e.g. BMNC, SLN111_D1) with transcriptomic counts in anndata.X
- Spatial coordinates stored in anndata.obsm['spatial'] for spatially resolved data
- Proteomic (e.g. ADT) count matrix as a separate AnnData object

## Outputs

- Log1p-normalized, HVG-selected transcriptomic AnnData with 200-component PCA reduction stored as 'X_pca'
- CLR-normalized proteomic AnnData
- Preprocessed AnnData objects ready for Cal_Nbrs_Net neighborhood graph construction

## How to apply

Load each omics layer as a separate AnnData object, verifying expression profiles are stored in anndata.X and, for spatial data, coordinates in anndata.obsm['spatial'] (otherwise the data is treated as single-cell). For the transcriptome, apply count-depth scaling with log1p transformation, select the top highly variable genes, and run PCA to reduce the profile to 200 dimensions with SCANPY; this reduced 'X_pca' matrix later feeds Cal_Nbrs_Net (feat='X_pca') for Euclidean-distance neighbor computation. For the proteome (e.g. ADT counts), apply centered log-ratio (CLR) transformation to normalize across antibody-derived tags. Store the preprocessed AnnData objects so downstream steps (Cal_Nbrs_Net, prune_net, ssgate.train) can consume them; save graphs to nbrs_net.h5ad per the repository workflow.

## Related tools

- **SCANPY** (Performs count-depth scaling with log1p transformation, highly variable gene selection, PCA to 200 dimensions, and CLR transformation of proteomics data)
- **anndata** (Required input format: expression profiles must be in anndata.X and spatial coordinates in anndata.obsm['spatial']) — https://anndata.readthedocs.io/en/latest/
- **SSGATE** (Downstream consumer of the preprocessed AnnData objects; PCA-reduced 'X_pca' features feed Cal_Nbrs_Net) — https://github.com/Linliu-Bioinf/SSGATE

## Examples

```
import scanpy as sc; sc.pp.normalize_total(adata); sc.pp.log1p(adata); sc.pp.highly_variable_genes(adata); sc.pp.pca(adata, n_comps=200)
```

## Evaluation signals

- Transcriptomic AnnData contains a PCA representation with exactly 200 dimensions stored as 'X_pca', matching the paper's dimensionality-reduction step
- Expression profiles are confirmed to reside in anndata.X before downstream functions; missing/incorrect storage means 'the data cannot be read correctly'
- For spatial inputs, anndata.obsm['spatial'] is populated; otherwise the object will silently be processed as single-cell multi-omics data
- Proteomics values are CLR-transformed (non-negative log-ratio scale) rather than raw counts
- Preprocessed objects pass into Cal_Nbrs_Net without error and the resulting adata.uns['nbrs_net'] is populated for both modalities

## Limitations

- Normalization and dimensionality reduction methods for integrating omics beyond transcriptome/proteome are not yet available, per the article's discussion
- Hyperparameters are set 'based on experience' as defaults rather than systematically derived
- If spatial coordinates are not in anndata.obsm['spatial'], the data is silently treated as single-cell multi-omics because spatial information cannot be read

## Evidence

- [readme] README data format requirements: [evidence span withheld — anchor status: not_found]
- [readme] README spatial storage requirement: "The spatial coordinates from spatial multi-omics data are required to be stored in anndata.obsm['spatial'], otherwise it will be processed as single-cell multi-omics data"
- [readme] Cal_Nbrs_Net feature parameter: "The feature matrix used to calculate the neighbors. Can be either reduced coordinates ('X_pca') or spatial coordinates ('spatial')."
- [full_text] Task card workflow (preprocessing details): [evidence span withheld — anchor status: not_found]
- [full_text] Task card PCA dimensionality: [evidence span withheld — anchor status: not_found]
