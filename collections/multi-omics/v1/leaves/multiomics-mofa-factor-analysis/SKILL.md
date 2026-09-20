---
name: multiomics-mofa-factor-analysis
description: Use when a MuData object contains observations measured across two or
  more modalities (RNA + ATAC multiome, or CITE-seq RNA + protein) and a joint, interpretable
  latent space or joint embedding is needed for visualisation and cell-type-level
  interpretation.
when_to_use_negative:
- The dataset is single-omics only (one modality) — plain scanpy/MOFA on a single
  AnnData suffices and MUON's multimodal intersection/integration machinery is unnecessary.
- The assay requires explicit genomic-coordinate or spatial-coordinate modelling,
  or modelling dependencies between feature sets across omics — MUON lists these as
  future development, not current capability.
- Modalities have largely disjoint observations and cell-to-cell correspondence is
  not meaningful — intersect_obs would discard too much data.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3366
tools:
- name: MUON
  role: 'Data standard and analysis framework: MuData storage, per-modality preprocessing
    (pp.tfidf, pp.dsb, pp.intersect_obs), and multimodal integration via mu.tl.mofa,
    mu.pp.neighbors, mu.tl.umap'
  repo: https://github.com/scverse/muon
- name: MOFA+
  role: Statistical engine behind mu.tl.mofa for unsupervised joint integration of
    multi-modal single-cell data, yielding factors stored in X_mofa
- name: SCANPY
  role: Per-modality RNA preprocessing (normalisation, PCA) and UMAP computation reused
    inside the MUON workflow
- name: pandas
  role: Exporting per-cell embedding coordinates (per-modality and joint UMAPs) to
    CSV tables for validation
- name: dsb
  role: 'Protein-count normalisation for CITE-seq prior to integration: pt.pp.dsb
    denoises ADT signal using empty-droplet background'
  repo: https://github.com/niaid/dsb
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:025b2cb04082559265e8babbd9298ccd57293fe9486196a2f1087a2d8c5b929f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/muon
  tools:
  - MOFA+
  - MUON
  - NumPy
  - SCANPY
  - dsb
  - pandas
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# multiomics-mofa-factor-analysis

## Summary

Fit multi-omics factor analysis (MOFA+) on a MuData object holding multiple modalities (e.g. RNA + ATAC or RNA + protein) to obtain an interpretable joint latent space (X_mofa), then build a weighted-nearest-neighbours cell graph and joint UMAP on top of it. This is MUON's core integration strategy for combining single-omics preprocessing with multimodal dimensionality reduction.

## When to use

Use when a MuData object contains observations measured across two or more modalities (RNA + ATAC multiome, or CITE-seq RNA + protein) and a joint, interpretable latent space or joint embedding is needed for visualisation and cell-type-level interpretation.

## When NOT to use

- The dataset is single-omics only (one modality) — plain scanpy/MOFA on a single AnnData suffices and MUON's multimodal intersection/integration machinery is unnecessary.
- The assay requires explicit genomic-coordinate or spatial-coordinate modelling, or modelling dependencies between feature sets across omics — MUON lists these as future development, not current capability.
- Modalities have largely disjoint observations and cell-to-cell correspondence is not meaningful — intersect_obs would discard too much data.

## Inputs

- MuData object with two or more modalities (e.g. 'rna' and 'atac' from a 10x multiome run, or 'rna' and 'prot' from CITE-seq feature-barcode matrices)
- Raw (unfiltered) Cell Ranger feature-barcode matrices (filtered_feature_bc_matrix.h5 and raw .h5) for DSB background estimation
- Per-cell annotations in mdata.obs for colouring/interpreting factors and embeddings

## Outputs

- mdata.obsm['X_mofa'] — the joint MOFA factor matrix per cell
- Joint 2D UMAP coordinates in mdata.obsm from the WNN neighbour graph
- Saved MuData file (e.g. mdata_integrated.h5mu) whose .mod group contains both modalities
- CSV export of per-cell embedding coordinates (e.g. RNA-PC UMAP, ATAC-PC UMAP, MOFA+WNN UMAP)

## How to apply

Load the dataset as a MuData object (e.g. with muon.read_10x_h5 for Cell Ranger feature-barcode matrices) and preprocess each modality separately: normalise RNA with scanpy, apply TF-IDF to ATAC with ac.pp.tfidf(atac, scale_factor=1e4), and normalise protein with the denoised-and-scaled-by-background method pt.pp.dsb(adata_prot, adata_prot_raw, empty_counts_range=...) for CITE-seq. Restrict the object to observations shared by all modalities with mu.pp.intersect_obs(mdata). Run mu.tl.mofa(mdata) with the default feature selection use_var='highly_variable', which stores the learned factors in mdata.obsm['X_mofa']. The rationale is MUON's modular design: existing single-omics methods handle per-modality preprocessing, while MOFA+ supplies the joint integration; downstream, build the multimodal WNN graph with mu.pp.neighbors(mdata) and embed it with mu.tl.umap(mdata). Annotate cells in mdata.obs (e.g. coarse-grained cell type, naive vs memory T cells) and colour the joint UMAP to attribute biological splits (such as the CD45 RA/RO distinction) to specific modalities and features.

## Related tools

- **MUON** (Data standard and analysis framework: MuData storage, per-modality preprocessing (pp.tfidf, pp.dsb, pp.intersect_obs), and multimodal integration via mu.tl.mofa, mu.pp.neighbors, mu.tl.umap) — https://github.com/scverse/muon
- **MOFA+** (Statistical engine behind mu.tl.mofa for unsupervised joint integration of multi-modal single-cell data, yielding factors stored in X_mofa)
- **SCANPY** (Per-modality RNA preprocessing (normalisation, PCA) and UMAP computation reused inside the MUON workflow)
- **pandas** (Exporting per-cell embedding coordinates (per-modality and joint UMAPs) to CSV tables for validation)
- **dsb** (Protein-count normalisation for CITE-seq prior to integration: pt.pp.dsb denoises ADT signal using empty-droplet background) — https://github.com/niaid/dsb

## Examples

```
mu.tl.mofa(mdata, use_var='highly_variable'); mu.pp.neighbors(mdata); mu.tl.umap(mdata); mdata.write('mdata_integrated.h5mu')
```

## Evaluation signals

- 'X_mofa' in mdata.obsm evaluates True after mu.tl.mofa completes with use_var='highly_variable'.
- UMAP coordinates computed from the WNN graph are present in mdata.obsm with exactly two finite numeric columns per embedding.
- The exported CSV table has one row per cell with two finite numeric UMAP coordinate columns for each embedding (RNA-PC, ATAC-PC, MOFA+WNN).
- The saved .h5mu file contains a .mod group with both modalities (e.g. 'rna' and 'prot'), confirming multimodal serialisation to HDF5.
- The joint UMAP, coloured by cell-type annotation in mdata.obs, separates expected populations (e.g. naive vs memory T cells attributed to CD45 RA/RO protein abundance).

## Limitations

- MOFA integration via mu.tl.mofa uses default feature selection use_var='highly_variable', so features outside the highly-variable sets are excluded from factor learning.
- Explicit support for genomic-coordinate based assays and assays with spatial coordinates is planned future development, not a current capability.
- Explicitly modelling dependencies between feature sets across omics, or between multiple sets of multi-omics experiments, is planned future work in MuData.
- Handling of novel, emerging multi-omics assays is future work rather than a current capability.

## Evidence

- [methods] In order to keep observations present in all the modalities, there is {func}`muon.pp.intersect_obs`.: "In order to keep observations present in all the modalities, there is {func}`muon.pp.intersect_obs`."
- [methods] TF-IDF transformation is implemented in the muon's ATAC module: "TF-IDF transformation is implemented in the muon's ATAC module:"
- [methods] pt.pp.dsb(adata_prot, adata_prot_raw, empty_counts_range=...): "pt.pp.dsb(adata_prot, adata_prot_raw, empty_counts_range=...)"
- [abstract] MOFA+: a statistical framework for comprehensive integration of multi-modal single-cell data: "MOFA+: a statistical framework for comprehensive integration of multi-modal single-cell data"
- [discussion] MUON is designed in a modular fashion, which means that existing methods and tools for processing individual omics can be reused: "MUON is designed in a modular fashion, which means that existing methods and tools for processing individual omics can be reused"
- [readme] `muon` is a multimodal omics Python framework.: "`muon` is a multimodal omics Python framework."
