---
name: pseudo-spatial-coordinate-generation-for-single-cell-multiomics
description: Use when applying a spatial model (e.g., SMOPCA) to single-cell multi-omics
  count matrices that have no physical coordinates and you need pseudo-spatial locations
  derived from the transcriptomic latent space.
when_to_use_negative:
- Data already carries real spatial coordinates (e.g., DLPFC, SLN111D1/D2, SLN208D1/D2,
  spatial-CITE-seq human tonsil GSE213264, Stereo-CITE-seq mouse thymus) — use the
  measured locations instead of UMAP-derived pseudo-coordinates
- The downstream method does not require a location input (e.g., purely single-cell
  clustering with BREM-SC, CiteFuse, Specter, or TotalVI)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3934
- http://edamontology.org/topic_3170
tools:
- name: Seurat V4
  role: performs normalization, HVG selection, PCA (top 50 PCs), and UMAP to produce
    the 2D pseudo-spatial coordinates
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:b9519be5dd220f5dbdffdaf9aabadca3ca9eb16d6b8ee533d9b0a18759a06d3b
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3934
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# pseudo-spatial-coordinate-generation-for-single-cell-multiomics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates 2D pseudo-spatial coordinates for single-cell multi-omics data (e.g., CITE-seq, SMAGE-seq) that lack real spatial positions, by running UMAP on the top 50 PCs of the top 2000 highly variable or mapped genes in Seurat V4, so that a spatially aware model like SMOPCA can accept them as its location input.

## When to use

Use when applying a spatial model (e.g., SMOPCA) to single-cell multi-omics count matrices that have no physical coordinates and you need pseudo-spatial locations derived from the transcriptomic latent space.

## When NOT to use

- Data already carries real spatial coordinates (e.g., DLPFC, SLN111D1/D2, SLN208D1/D2, spatial-CITE-seq human tonsil GSE213264, Stereo-CITE-seq mouse thymus) — use the measured locations instead of UMAP-derived pseudo-coordinates
- The downstream method does not require a location input (e.g., purely single-cell clustering with BREM-SC, CiteFuse, Specter, or TotalVI)

## Inputs

- preprocessed single-cell multi-omics count matrix (e.g., CITE-seq gene expression matrix; SMAGE-seq gene activity matrix collapsed from ATAC peaks)
- feature selection: top 2000 highly variable genes (CITE-seq) or mapped genes (SMAGE-seq)

## Outputs

- umap_coordinates.csv: 2D pseudo-spatial location matrix with one UMAP (x, y) coordinate pair per cell

## How to apply

Load the preprocessed single-cell multi-omics count matrix and normalize by library size (total read count), then log-transform and scale to zero mean and unit variance. Select the top 2000 highly variable genes for CITE-seq data, or the mapped genes of the gene activity matrix for SMAGE-seq data (after collapsing the ATAC peak matrix into a gene activity matrix following the Satija lab protocol), as input features. Run PCA in Seurat V4 and use the top 50 PCs as input to UMAP to compute the 2D embedding. Extract the 2D UMAP coordinates per cell and write them as the pseudo-spatial location matrix (umap_coordinates.csv). The rationale is that UMAP's nonlinear neighborhood structure approximates tissue-like similarity, giving SMOPCA the location input its spatial kernel requires.

## Related tools

- **Seurat V4** (performs normalization, HVG selection, PCA (top 50 PCs), and UMAP to produce the 2D pseudo-spatial coordinates) — https://github.com/satijalab/seurat

## Examples

```
R (Seurat): pbmc <- NormalizeData(pbmc); pbmc <- FindVariableFeatures(pbmc, nfeatures = 2000); pbmc <- ScaleData(pbmc); pbmc <- RunPCA(pbmc, npcs = 50); pbmc <- RunUMAP(pbmc, dims = 1:50); write.csv(Embeddings(pbmc, 'umap'), 'umap_coordinates.csv')
```

## Evaluation signals

- umap_coordinates.csv contains exactly one 2D coordinate row per cell, matching the cell order/count of the input matrix (e.g., 3762 cells for the PBMC CITE-seq dataset)
- Input features are restricted to the top 2000 highly variable genes (CITE-seq) or mapped genes (SMAGE-seq) selected after normalization
- The UMAP embedding was computed from the top 50 principal components, not from the full feature matrix
- SMOPCA run with these pseudo-coordinates yields clustering labels comparable to those from non-spatial baselines (e.g., BREM-SC, Specter, TotalVI) on the same dataset

## Limitations

- UMAP-derived pseudo-spatial structure is a surrogate for real tissue geometry; SMOPCA's learned lengthscale parameter may be difficult to fit when the data are sparse and noisy or the pseudo-spatial structure is complicated
- The downstream SMOPCA analysis uses normalized data rather than raw counts and does not model dropout or the mean-variance relationship, which may reduce analytical power
- The source card workflow specifies CITE-seq and SMAGE-seq feature definitions; other modality combinations may need different gene-selection criteria

## Evidence

- [full_text] workflow card, step 5: [evidence span withheld — anchor status: not_found]
- [full_text] workflow card, steps 2-4: [evidence span withheld — anchor status: not_found]
- [discussion] article discussion: [evidence span withheld — anchor status: not_found]
