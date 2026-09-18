---
name: highly-variable-gene-selection-cite-seq
description: Use when single-cell multi-omics data (e.g. CITE-seq PBMC or SMAGE-seq
  PBMC3K/PBMC10K) lacks spatial coordinates and you need pseudo-spatial locations
  for a spatially aware model like SMOPCA.
when_to_use_negative:
- Data already has true spatial coordinates (e.g. DLPFC 12 sections, spatial-CITE-seq
  human tonsil GSE213264, Stereo-CITE-seq mouse thymus) — use the real locations instead
  of UMAP pseudo-coordinates
- Analysis goal is modeling raw count distributions (dropouts, mean–variance relationship),
  since SMOPCA and this UMAP pipeline operate on normalized, scaled data rather than
  raw counts
- Input is an unnormalized peak matrix — peaks must first be collapsed into a gene
  activity matrix per the Satija lab protocol before gene selection
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat V4
  role: Performs normalization, selection of top 2000 highly variable genes, PCA on
    selected features, and UMAP on the top 50 PCs to produce the 2D pseudo-spatial
    coordinates
  repo: https://github.com/satijalab/seurat
- name: BREM-SC
  role: Listed among the tools surfaced with the pseudo-spatial UMAP filter workflow
    for single-cell multi-omics clustering comparison
  repo: https://github.com/tarot0410/BREMSC
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:cb71d81592249625f11953cd909c91cd74cb763dd408a3f57cdef56753b5d2c0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hemberg-lab/SC3
  repo_url: https://github.com/satijalab/seurat
  tools:
  - BREM-SC
  - Seurat V4
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Highly variable gene selection for CITE-seq pseudo-spatial UMAP generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects the top 2000 highly variable genes (CITE-seq) or mapped genes (SMAGE-seq) from normalized single-cell multi-omics matrices as features for PCA followed by a 2D UMAP embedding in Seurat V4, whose coordinates serve as pseudo-spatial input locations for SMOPCA when true spatial coordinates are absent.

## When to use

Use when single-cell multi-omics data (e.g. CITE-seq PBMC or SMAGE-seq PBMC3K/PBMC10K) lacks spatial coordinates and you need pseudo-spatial locations for a spatially aware model like SMOPCA.

## When NOT to use

- Data already has true spatial coordinates (e.g. DLPFC 12 sections, spatial-CITE-seq human tonsil GSE213264, Stereo-CITE-seq mouse thymus) — use the real locations instead of UMAP pseudo-coordinates
- Analysis goal is modeling raw count distributions (dropouts, mean–variance relationship), since SMOPCA and this UMAP pipeline operate on normalized, scaled data rather than raw counts
- Input is an unnormalized peak matrix — peaks must first be collapsed into a gene activity matrix per the Satija lab protocol before gene selection

## Inputs

- Preprocessed single-cell multi-omics count matrix (e.g. CITE-seq PBMC: 3762 cells, 33,538 genes, 49 proteins)
- SMAGE-seq gene activity matrix obtained by mapping ATAC reads to gene regions (e.g. PBMC3K ~3000 cells, PBMC10K ~10,000 cells)

## Outputs

- Matrix of top 2000 highly variable genes (CITE-seq) or mapped genes (SMAGE-seq) as selected input features
- 2D UMAP pseudo-spatial coordinate matrix for every cell (umap_coordinates.csv)

## How to apply

First normalize the count matrix by library size (total read count), apply log transformation, and scale to zero mean and unit variance. Then select the top 2000 highly variable genes for CITE-seq data, or the mapped genes (ATAC reads collapsed into a gene activity matrix) for SMAGE-seq data, as input features. Run PCA on these features in Seurat V4 and feed the top 50 principal components into UMAP to compute the 2D embedding. Extract the 2D UMAP coordinates for every cell and write them as the pseudo-spatial location matrix (e.g. umap_coordinates.csv), which becomes SMOPCA's location input. The rationale is that UMAP's nonlinear neighborhood structure approximates the spatial correlation structure that SMOPCA's kernel matrix expects, allowing the spatially aware model to be applied to non-spatial single-cell data.

## Related tools

- **Seurat V4** (Performs normalization, selection of top 2000 highly variable genes, PCA on selected features, and UMAP on the top 50 PCs to produce the 2D pseudo-spatial coordinates) — https://github.com/satijalab/seurat
- **BREM-SC** (Listed among the tools surfaced with the pseudo-spatial UMAP filter workflow for single-cell multi-omics clustering comparison) — https://github.com/tarot0410/BREMSC

## Examples

```
pbmc <- NormalizeData(pbmc); pbmc <- FindVariableFeatures(pbmc, nfeatures = 2000); pbmc <- ScaleData(pbmc); pbmc <- RunPCA(pbmc); pbmc <- RunUMAP(pbmc, dims = 1:50)
```

## Evaluation signals

- The feature matrix contains exactly the top 2000 highly variable genes (CITE-seq) or the full set of mapped genes (SMAGE-seq), selected after library-size normalization, log transformation, and zero-mean/unit-variance scaling
- UMAP is computed on exactly the top 50 principal components, not fewer or more
- The output umap_coordinates.csv has one row per cell with 2D coordinates matching cell identifiers of the input count matrix
- Feeding the UMAP coordinates into SMOPCA yields latent representations that preserve neighboring similarity, consistent with the pseudo-spatial design
- Gene/protein features with zero counts are filtered out prior to feature selection

## Limitations

- UMAP coordinates are pseudo-spatial proxies, not real tissue locations; their fidelity to genuine spatial structure depends on the UMAP neighborhood geometry
- SMOPCA uses normalized data rather than raw counts, potentially causing a loss of inference accuracy and reduced analytical power (dropout events and mean–variance relationships are not modeled)
- The top 50 PCs / 2000 HVGs configuration is dataset-specific (tuned for CITE-seq PBMC and SMAGE-seq PBMC3K/PBMC10K); other datasets may need recalibration

## Evidence

- [discussion] HVG selection for PCA/UMAP input: [evidence span withheld — anchor status: not_found]
# choose variable features
pbmc <- FindVariableFeatures(pbmc)"
