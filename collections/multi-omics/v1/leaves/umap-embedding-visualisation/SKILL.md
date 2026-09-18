---
name: umap-embedding-visualisation
description: Use when a MuData object contains per-modality PCs or joint latent spaces
  (X_mofa, WNN graph) and you need 2D UMAP embeddings per modality and jointly, to
  visualise cell-type structure across modalities.
when_to_use_negative:
- The data are single-omics with no second modality — plain scanpy UMAP on one AnnData
  suffices and the WNN/MOFA joint graph is not meaningful.
- Only raw counts are available without per-modality preprocessing (RNA normalisation,
  ATAC TF-IDF) — UMAP on unprocessed counts will not reflect the paper's embeddings.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3611
tools:
- name: MUON
  role: provides mu.pp.neighbors for the weighted-nearest-neighbours cell graph across
    modalities and mu.tl.umap for the joint 2D embedding
  repo: https://github.com/scverse/muon
- name: SCANPY
  role: normalises RNA, computes per-modality principal components, and runs scanpy
    UMAP for the individual-modality embeddings
- name: MOFA+
  role: learns the joint latent space (X_mofa in mdata.obsm) that the joint UMAP is
    built on
- name: pandas
  role: exports per-cell embedding coordinates to a CSV table
- name: NumPy
  role: supports numerical handling of embedding coordinates during validation
merged_aliases:
- umap-embedding-visualisation-multimodal
merged_alias_records:
- alias: umap-embedding-visualisation-multimodal
  slug: umap-embedding-visualisation-multimodal
  jaccard_score: 0.75
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:890e771bbdcce3afa764143924524f99191c145043ad28045554afad4b4b14ca
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3611
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2222222222222222
n_claims: 9
---

# umap-embedding-visualisation

## Summary

Generates and stores 2D UMAP embeddings of multimodal (e.g. PBMC multiome / CITE-seq) MuData objects, both from individual-modality principal components and from joint multi-omics representations such as MOFA factors or the MUON WNN cell-neighbour graph, with cells coloured by coarse-grained cell type.

## When to use

Use when a MuData object contains per-modality PCs or joint latent spaces (X_mofa, WNN graph) and you need 2D UMAP embeddings per modality and jointly, to visualise cell-type structure across modalities.

## When NOT to use

- The data are single-omics with no second modality — plain scanpy UMAP on one AnnData suffices and the WNN/MOFA joint graph is not meaningful.
- Only raw counts are available without per-modality preprocessing (RNA normalisation, ATAC TF-IDF) — UMAP on unprocessed counts will not reflect the paper's embeddings.

## Inputs

- MuData object with 'rna' and 'atac' modalities (PBMC multiome, e.g. loaded via mu.read) or 'rna' and 'prot' modalities (CITE-seq via muon.read_10x_h5)
- Per-modality PCA/LSI principal component representations in each AnnData modality
- mdata.obsm['X_mofa'] factors from mu.tl.mofa
- mdata.obs coarse-grained cell type annotations for colouring

## Outputs

- 2D UMAP coordinates per embedding stored in mdata.obsm (RNA-PC UMAP, ATAC-PC UMAP, MOFA+WNN UMAP)
- UMAP plots coloured by coarse-grained cell type (Fig. 3b style)
- CSV table with one row per cell and exactly two finite numeric UMAP coordinate columns per embedding

## How to apply

Preprocess each modality separately first — normalise RNA with scanpy and apply TF-IDF to ATAC with ac.pp.tfidf(scale_factor=1e4) — then compute principal components per modality and run scanpy UMAP to obtain 2D embeddings from RNA PCs and from ATAC (LSI) PCs. For the joint view, fit multi-omic factor analysis with mu.tl.mofa using the default feature selection use_var='highly_variable', which stores factors in mdata.obsm['X_mofa']; build the weighted-nearest-neighbours cell graph across modalities with mu.pp.neighbors and embed it with mu.tl.umap. Annotate cells with coarse-grained cell type labels in mdata.obs and colour all UMAP plots by this annotation so per-modality and joint embeddings can be compared directly. Export the per-cell embedding coordinates (RNA-PC UMAP, ATAC-PC UMAP, MOFA+WNN UMAP) to a CSV table for downstream checks.

## Related tools

- **MUON** (provides mu.pp.neighbors for the weighted-nearest-neighbours cell graph across modalities and mu.tl.umap for the joint 2D embedding) — https://github.com/scverse/muon
- **SCANPY** (normalises RNA, computes per-modality principal components, and runs scanpy UMAP for the individual-modality embeddings)
- **MOFA+** (learns the joint latent space (X_mofa in mdata.obsm) that the joint UMAP is built on)
- **pandas** (exports per-cell embedding coordinates to a CSV table)
- **NumPy** (supports numerical handling of embedding coordinates during validation)

## Examples

```
import muon as mu; mu.pp.neighbors(mdata); mu.tl.umap(mdata)
```

## Evaluation signals

- The output CSV exists and contains one row per cell with exactly two finite numeric UMAP coordinate columns per embedding (RNA-PC UMAP, ATAC-PC UMAP, MOFA+WNN UMAP).
- 'X_mofa' is present in mdata.obsm after mu.tl.mofa.
- UMAP coordinates are present in mdata.obsm after mu.pp.neighbors and mu.tl.umap.
- Per-modality UMAPs and the joint UMAP, coloured by the same coarse-grained cell type annotation, show concordant cell-type structure as in Fig. 3b.
- In the CITE-seq case, the saved .h5mu file contains the .mod group with both the 'rna' and 'prot' modalities.

## Limitations

- MOFA feature selection depends on the default use_var='highly_variable' setting; without prior per-modality HVG selection the joint embedding quality degrades.
- Observations missing from any modality must be reconciled first (mu.pp.intersect_obs) or the joint neighbour graph is ill-defined.
- The embeddings reflect the graph/factors fed in; UMAP itself does not attribute biological splits such as the naive-vs-memory T cell CD45 RA/RO distinction — that attribution comes from the protein-level features.

## Evidence

- [results] MUON generates, stores, and operates with multiple embeddings built from individual modalities (Fig. 3b, left) or jointly for both modalities based on the MOFA factors or the WNN graph (Fig. 3b, right), with cells coloured by coarse-grained cell type.: [evidence span withheld — anchor status: not_found]
- [methods] TF-IDF transformation is implemented in the muon's ATAC module: ac.pp.tfidf(atac, scale_factor=1e4): "TF-IDF transformation is implemented in the muon's ATAC module"
- [methods] In order to keep observations present in all the modalities, there is {func}`muon.pp.intersect_obs`.: "In order to keep observations present in all the modalities, there is {func}`muon.pp.intersect_obs`."
- [readme] `muon` is a multimodal omics Python framework.: "`muon` is a multimodal omics Python framework."
