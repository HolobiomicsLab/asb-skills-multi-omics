---
name: weighted-nearest-neighbour-graph-construction
description: Use when a MuData object with per-modality representations (e.g. PCA
  on RNA/ATAC PCs or MOFA factors in mdata.obsm['X_mofa']) is available and a joint
  cell-cell neighbour graph or UMAP combining modalities is needed.
when_to_use_negative:
- Only a single modality is present — per-modality PCA + scanpy UMAP alone is sufficient
  and WNN across modalities has nothing to weight
- Modalities contain different cells and intersection is not desired — mu.pp.intersect_obs
  would drop observations, so use modality-specific embeddings instead
- The goal is modality-specific QC (e.g. nucleosome signal, TSS enrichment for ATAC)
  rather than joint integration
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: muon
  role: Provides mu.pp.neighbors to build the multimodal weighted-nearest-neighbours
    graph and mu.tl.umap to embed it; also supplies mu.pp.intersect_obs to keep shared
    cells across modalities
  repo: https://github.com/scverse/muon
- name: MOFA+
  role: Fits multi-omic factor analysis (mu.tl.mofa) whose factors in mdata.obsm['X_mofa']
    serve as the joint representation from which the WNN graph is built
- name: scanpy
  role: Normalises RNA (and per-modality PCA/UMAP for individual-modality embeddings)
    before joint graph construction
- name: NumPy
  role: Numeric backend for validating that UMAP coordinate columns are finite numeric
    values
- name: pandas
  role: Exports per-cell embedding coordinates (RNA-PC UMAP, ATAC-PC UMAP, MOFA+WNN
    UMAP) to a CSV table
merged_aliases:
- weighted-nearest-neighbours-graph-construction
merged_alias_records:
- alias: weighted-nearest-neighbours-graph-construction
  slug: weighted-nearest-neighbours-graph-construction
  jaccard_score: 0.6667
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:d09873a2e113de244ba93666ac7384c98354f75c1357a1580a0c15c538143dc7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/muon
  tools:
  - MOFA+
  - MUON
  - NumPy
  - SCANPY
  - muon
  - pandas
  - scanpy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# weighted-nearest-neighbour-graph-construction

## Summary

Builds a joint weighted-nearest-neighbours (WNN) cell graph across modalities with mu.pp.neighbors in MUON and embeds it with mu.tl.umap, yielding a multimodal 2D embedding (e.g. of PBMC CITE-seq or multiome data) that integrates RNA, protein, and ATAC views of each cell.

## When to use

Use when a MuData object with per-modality representations (e.g. PCA on RNA/ATAC PCs or MOFA factors in mdata.obsm['X_mofa']) is available and a joint cell-cell neighbour graph or UMAP combining modalities is needed.

## When NOT to use

- Only a single modality is present — per-modality PCA + scanpy UMAP alone is sufficient and WNN across modalities has nothing to weight
- Modalities contain different cells and intersection is not desired — mu.pp.intersect_obs would drop observations, so use modality-specific embeddings instead
- The goal is modality-specific QC (e.g. nucleosome signal, TSS enrichment for ATAC) rather than joint integration

## Inputs

- MuData object with per-modality AnnData modalities (e.g. 'rna' and 'prot' from CITE-seq, or 'rna' and 'atac' from multiome) loaded via mu.read or muon.read_10x_h5
- Per-modality low-dimensional representations: PCA coordinates for RNA/ATAC or MOFA factors stored in mdata.obsm['X_mofa']
- Optional cell type annotations in mdata.obs for colouring the joint UMAP

## Outputs

- Weighted-nearest-neighbours cell graph stored in the MuData object (connectivities/distances in mdata.obsp)
- Joint 2D UMAP coordinates in mdata.obsm
- Persisted multimodal object, e.g. mdata_integrated.h5mu, containing the .mod group with both modalities

## How to apply

After per-modality preprocessing (scanpy normalisation for RNA, TF-IDF with scale_factor=1e4 for ATAC via ac.pp.tfidf, or dsb normalisation for protein via pt.pp.dsb) and joint factor analysis with mu.tl.mofa using the default feature selection use_var='highly_variable', call mu.pp.neighbors on the MuData object to build a weighted-nearest-neighbours cell graph across modalities, then compute the joint 2D embedding with mu.tl.umap. Restrict to shared cells first with mu.pp.intersect_obs when modalities have different observation sets, e.g. intersecting 'rna' and 'prot' modalities in CITE-seq. The resulting embedding supports downstream tasks such as annotating naive vs memory T cells and visualising CD45 RA/RO protein levels on the joint UMAP. Persist results with mdata.write('mdata_integrated.h5mu') so the .h5mu file contains the .mod group with all modalities.

## Related tools

- **muon** (Provides mu.pp.neighbors to build the multimodal weighted-nearest-neighbours graph and mu.tl.umap to embed it; also supplies mu.pp.intersect_obs to keep shared cells across modalities) — https://github.com/scverse/muon
- **MOFA+** (Fits multi-omic factor analysis (mu.tl.mofa) whose factors in mdata.obsm['X_mofa'] serve as the joint representation from which the WNN graph is built)
- **scanpy** (Normalises RNA (and per-modality PCA/UMAP for individual-modality embeddings) before joint graph construction)
- **NumPy** (Numeric backend for validating that UMAP coordinate columns are finite numeric values)
- **pandas** (Exports per-cell embedding coordinates (RNA-PC UMAP, ATAC-PC UMAP, MOFA+WNN UMAP) to a CSV table)

## Examples

```
mu.pp.neighbors(mdata); mu.tl.umap(mdata); mdata.write('mdata_integrated.h5mu')
```

## Evaluation signals

- The check '"X_mofa" in mdata.obsm' evaluates True before graph construction
- UMAP coordinates are present in mdata.obsm with exactly two finite numeric coordinate columns per embedding and one row per cell
- The saved .h5mu file contains the .mod group with both the 'rna' and 'prot' (or 'atac') modalities
- Cells coloured by coarse-grained cell type annotation in mdata.obs show biologically coherent structure, e.g. naive vs memory T cells separated and attributed to CD45 RA/RO protein abundance on the joint UMAP

## Limitations

- Joint graph construction requires observations present in all modalities; mu.pp.intersect_obs must be applied, which drops cells missing from any modality
- MOFA feature selection defaults to use_var='highly_variable'; without prior per-modality preprocessing (normalised RNA, TF-IDF ATAC with scale_factor=1e4, or dsb-corrected protein) the joint embedding quality degrades
- Emerging multi-omics assays and explicit support for genomic-coordinate or spatial-coordinate assays are listed as future development, not current capability

## Evidence

- [methods] In order to keep observations present in all the modalities, there is {func}`muon.pp.intersect_obs`.: "In order to keep observations present in all the modalities, there is {func}`muon.pp.intersect_obs`."
- [abstract] MUON enables a versatile range of analyses, from data preprocessing to flexible multi-omics alignment.: "MUON enables a versatile range of analyses, from data preprocessing to flexible multi-omics alignment"
- [readme] `muon` is a multimodal omics Python framework.: "`muon` is a multimodal omics Python framework."
- [methods] TF-IDF transformation is implemented in the muon's ATAC module: ac.pp.tfidf(atac, scale_factor=1e4): "ac.pp.tfidf(atac, scale_factor=1e4)"
- [discussion] the software facilitates combining single-omics analysis methods with a growing spectrum of multi-omics integration strategies: "the software facilitates combining single-omics analysis methods with a growing spectrum of multi-omics integration strategies"
