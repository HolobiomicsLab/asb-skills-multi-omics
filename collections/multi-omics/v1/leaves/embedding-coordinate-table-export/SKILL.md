---
name: embedding-coordinate-table-export
description: Use when a MuData object with RNA and ATAC modalities needs comparable
  2D latent-space visualisations built from individual-modality PCs as well as joint
  multi-omics representations such as MOFA factors or the WNN graph, with coordinates
  exported per cell.
when_to_use_negative:
- 'The data are unimodal single-omics (e.g. only scRNA-seq): joint multi-omics integration
  via MOFA factors or the WNN graph is not applicable, and standard scanpy PCA/UMAP
  suffices.'
- 'The input is raw sequencing output (FASTQ or fragment files) rather than a count
  matrix: preprocessing to a cell-by-feature matrix (e.g. via mu.read_10x_h5 or ac.read_snap)
  is required first.'
- The goal is filtering cells or features rather than embedding — use mu.pp.filter_obs
  / mu.pp.filter_var / mu.pp.intersect_obs instead.
edam_operation: http://edamontology.org/operation_0337
tools:
- name: MUON
  role: Provides the MuData container and the multi-omics API (mu.tl.mofa, mu.pp.neighbors,
    mu.tl.umap, ac.pp.tfidf) used to build and store the per-modality and joint embeddings
  repo: https://github.com/scverse/muon
- name: MOFA+
  role: Multi-omic factor analysis backend fitted via mu.tl.mofa with use_var='highly_variable'
    to yield X_mofa factors for the joint embedding
- name: SCANPY
  role: Performs RNA normalisation, per-modality PCA, and per-modality UMAP inside
    the MuData workflow
- name: pandas
  role: Used to write the exported per-cell embedding coordinate table to CSV
- name: NumPy
  role: Numeric validation of exported coordinate columns (finiteness, numeric dtype,
    two columns per embedding)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:bd351a131d823b05b2fc5f4866bd745f948b1f7dc4828c4cd42978db4b75fb64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/muon
  tools:
  - MOFA+
  - MUON
  - NumPy
  - SCANPY
  - pandas
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# embedding-coordinate-table-export

## Summary

Generates and stores 2D UMAP embeddings of a multimodal (e.g. PBMC multiome) MuData object from per-modality principal components (RNA PCs, ATAC LSI PCs) and from joint representations (MOFA factors, WNN cell-neighbour graph), then exports the per-cell coordinates to a CSV table for validation and reuse.

## When to use

Use when a MuData object with RNA and ATAC modalities needs comparable 2D latent-space visualisations built from individual-modality PCs as well as joint multi-omics representations such as MOFA factors or the WNN graph, with coordinates exported per cell.

## When NOT to use

- The data are unimodal single-omics (e.g. only scRNA-seq): joint multi-omics integration via MOFA factors or the WNN graph is not applicable, and standard scanpy PCA/UMAP suffices.
- The input is raw sequencing output (FASTQ or fragment files) rather than a count matrix: preprocessing to a cell-by-feature matrix (e.g. via mu.read_10x_h5 or ac.read_snap) is required first.
- The goal is filtering cells or features rather than embedding — use mu.pp.filter_obs / mu.pp.filter_var / mu.pp.intersect_obs instead.

## Inputs

- MuData object containing paired single-cell RNA and ATAC modalities (e.g. PBMC multiome dataset, loadable via mu.read or mu.read_10x_h5)
- Coarse-grained cell type annotations stored in mdata.obs for colouring the UMAP plots

## Outputs

- 2D UMAP embeddings stored per modality and jointly (e.g. in mdata.obsm: RNA-PC UMAP, ATAC-PC UMAP, joint MOFA+WNN UMAP)
- X_mofa factor matrix in mdata.obsm from mu.tl.mofa
- CSV table with per-cell embedding coordinates (one row per cell, two finite numeric UMAP coordinate columns per embedding)

## How to apply

Load the PBMC multiome dataset as a MuData object with mu.read, then preprocess each modality separately: normalise RNA with scanpy and apply TF-IDF to ATAC with ac.pp.tfidf(scale_factor=1e4). Compute principal components per modality with scanpy and run scanpy UMAP to obtain 2D embeddings from RNA PCs and from ATAC (LSI) PCs — these are the individual-modality embeddings (Fig. 3b, left). For joint embeddings, fit multi-omic factor analysis with mu.tl.mofa using the default feature selection use_var='highly_variable' (factors are stored as X_mofa in mdata.obsm), and/or build a weighted-nearest-neighbours cell graph across modalities with mu.pp.neighbors followed by mu.tl.umap to obtain the joint UMAP (Fig. 3b, right). Annotate cells with coarse-grained cell type labels in mdata.obs and colour the UMAP plots by this annotation. Finally export the per-cell embedding coordinates (RNA-PC UMAP, ATAC-PC UMAP, MOFA+WNN UMAP) to a CSV table with one row per cell.

## Related tools

- **MUON** (Provides the MuData container and the multi-omics API (mu.tl.mofa, mu.pp.neighbors, mu.tl.umap, ac.pp.tfidf) used to build and store the per-modality and joint embeddings) — https://github.com/scverse/muon
- **MOFA+** (Multi-omic factor analysis backend fitted via mu.tl.mofa with use_var='highly_variable' to yield X_mofa factors for the joint embedding)
- **SCANPY** (Performs RNA normalisation, per-modality PCA, and per-modality UMAP inside the MuData workflow)
- **pandas** (Used to write the exported per-cell embedding coordinate table to CSV)
- **NumPy** (Numeric validation of exported coordinate columns (finiteness, numeric dtype, two columns per embedding))

## Examples

```
mu.pp.neighbors(mdata); mu.tl.umap(mdata)
```

## Evaluation signals

- The exported CSV table exists and contains exactly one row per cell of the MuData object.
- Each embedding contributes exactly two finite numeric coordinate columns (no NaN/inf) in the CSV.
- 'X_mofa' is present in mdata.obsm after mu.tl.mofa, confirming MOFA factors were fitted and stored.
- UMAP plots coloured by the coarse-grained cell type annotation in mdata.obs show coherent cell-type structure, matching Fig. 3b (per-modality left, joint right).
- Both per-modality (RNA-PC, ATAC-PC) and joint (MOFA+WNN) embeddings are present and can be operated on within the MuData object.

## Limitations

- MOFA fitting depends on feature selection (default use_var='highly_variable'); features not flagged as highly variable are excluded from factor analysis.
- Joint embeddings (MOFA factors, WNN graph) require paired modalities with aligned observations; mu.pp.intersect_obs may be needed to keep observations present in all modalities.
- MUON's future-work items (explicit genomic-coordinate or spatial-coordinate assay support, modelling dependencies between feature sets across omics) are not available, so embeddings rely on the current per-modality/joint integration strategies.

## Evidence

- [results] MUON generates, stores, and operates with multiple embeddings built from individual modalities or jointly for both modalities based on the MOFA factors or the WNN graph: [evidence span withheld — anchor status: not_found]
- [methods] TF-IDF transformation is implemented in the muon's ATAC module with scale_factor=1e4: "TF-IDF transformation is implemented in the muon's ATAC module:

ac.pp.tfidf(atac, scale_factor=1e4)"
- [abstract] we present a data standard and an analysis framework for multi-omics, MUON, designed to organise, analyse, visualise, and exchange multimodal data: "we present a data standard and an analysis framework for multi-omics, MUON, designed to organise, analyse, visualise, and exchange multimodal data"
- [readme] muon is a multimodal omics Python framework (README): "`muon` is a multimodal omics Python framework."
