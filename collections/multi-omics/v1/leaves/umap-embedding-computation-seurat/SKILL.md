---
name: umap-embedding-computation-seurat
description: Use when single-cell multi-omics data lack true spatial coordinates but
  a spatially aware model (e.g., SMOPCA) requires a 2D location input per cell.
when_to_use_negative:
- The dataset already has real physical spatial coordinates (e.g., DLPFC SRT sections
  or Stereo-CITE-seq mouse thymus spots) — no pseudo-coordinates are needed.
- The analysis goal is purely visualization or cell-type clustering with non-spatial
  methods (e.g., Specter, BREM-SC), where a spatial location input adds nothing.
- Raw unnormalized counts are required downstream (e.g., count-aware models like TotalVI/scVI),
  since this skill operates on library-size-normalized, log-transformed, scaled data.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat V4
  role: Performs normalization, scaling, PCA on the selected features, and RunUMAP
    on the top 50 PCs to produce the 2D pseudo-spatial embedding.
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:128b68cfb4aa6cbacabecd54b3b38defe0bb4e554a14cd00dca957e2785f1b1a
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
  - Seurat V4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# umap-embedding-computation-seurat

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates 2D pseudo-spatial coordinates for single-cell multi-omics data (CITE-seq or SMAGE-seq) by running PCA on selected features in Seurat V4 and computing a 2D UMAP embedding from the top 50 PCs, so that spatially aware models such as SMOPCA can be applied to non-spatial data.

## When to use

Use when single-cell multi-omics data lack true spatial coordinates but a spatially aware model (e.g., SMOPCA) requires a 2D location input per cell.

## When NOT to use

- The dataset already has real physical spatial coordinates (e.g., DLPFC SRT sections or Stereo-CITE-seq mouse thymus spots) — no pseudo-coordinates are needed.
- The analysis goal is purely visualization or cell-type clustering with non-spatial methods (e.g., Specter, BREM-SC), where a spatial location input adds nothing.
- Raw unnormalized counts are required downstream (e.g., count-aware models like TotalVI/scVI), since this skill operates on library-size-normalized, log-transformed, scaled data.

## Inputs

- Preprocessed single-cell multi-omics count matrix (e.g., PBMC CITE-seq: 3762 cells, 33,538 genes, 49 proteins)
- Gene activity matrix for SMAGE-seq data after mapping ATAC reads to gene regions
- Cell barcode / feature annotation table linking cells to modalities

## Outputs

- 2D pseudo-spatial UMAP coordinate matrix with one (x, y) location per cell (e.g., umap_coordinates.csv)
- Location matrix formatted as SMOPCA spatial location input

## How to apply

Load the preprocessed single-cell multi-omics count matrix and normalize by library size (total read count), followed by log transformation and scaling to zero mean and unit variance. Select the top 2000 highly variable genes for CITE-seq data, or mapped genes for SMAGE-seq data (ATAC peaks collapsed into a gene activity matrix), as input features. Run PCA on the selected features with Seurat V4, then use the top 50 principal components (PCs) as input to UMAP to compute the 2D embedding in Seurat V4. Extract the 2D UMAP coordinates for every cell and write them as the pseudo-spatial location matrix (e.g., umap_coordinates.csv) to serve as SMOPCA's location input, substituting for physically measured spot positions.

## Related tools

- **Seurat V4** (Performs normalization, scaling, PCA on the selected features, and RunUMAP on the top 50 PCs to produce the 2D pseudo-spatial embedding.) — https://github.com/satijalab/seurat

## Examples

```
pbmc <- NormalizeData(pbmc); pbmc <- ScaleData(pbmc); pbmc <- RunPCA(pbmc, verbose = FALSE); pbmc <- RunUMAP(pbmc, dims = 1:50)
```

## Evaluation signals

- The output CSV contains exactly 2 coordinate columns and one row per cell, matching the input cell count (e.g., 3762 for the PBMC CITE-seq dataset).
- The UMAP was computed from exactly the top 50 PCs, and PCA was run on exactly the top 2000 highly variable genes (CITE-seq) or mapped genes (SMAGE-seq).
- Input features were normalized by library size, log-transformed, and scaled to zero mean and unit variance before PCA.
- Downstream SMOPCA clustering/spatial domain detection on the pseudo-coordinates runs without dimension mismatch between cells and location rows.
- Re-running the UMAP with a fixed seed reproduces a consistent neighborhood structure comparable to published PBMC results.

## Limitations

- UMAP coordinates are pseudo-spatial, not true tissue positions; the resulting 'spatial' structure reflects transcriptomic similarity and may not capture genuine tissue architecture.
- SMOPCA's lengthscale parameter may be hard to learn when spatial (or pseudo-spatial) structure is complicated or data are sparse and noisy, potentially requiring manual tuning.
- SMOPCA consumes normalized (not raw count) data, which the authors note can cause loss of inference accuracy and reduced analytical power, and it does not model dropout or the mean–variance relationship of counts.

## Evidence

- [discussion] we normalized the gene expression matrix and selected the top 2000 highly variable genes (CITE-seq data) or mapped genes (SMAGE-seq data) as input features: [evidence span withheld — anchor status: not_found]
- [readme] # Run PCA, select 20 PCs for tSNE visualization and graph-based clustering
pbmc <- RunPCA(pbmc, verbose = FALSE): [evidence span withheld — anchor status: not_found]
