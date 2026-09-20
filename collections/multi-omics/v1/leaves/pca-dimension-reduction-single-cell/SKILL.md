---
name: pca-dimension-reduction-single-cell
description: Use when a single-cell CITE-seq or SMAGE-seq count matrix needs a linear
  low-dimensional embedding before UMAP visualization, clustering, or as SMOPCA's
  downstream input.
when_to_use_negative:
- Input is already a low-dimensional embedding or PC matrix — re-running PCA adds
  no value
- The goal is spatially aware dimension reduction on data with real spatial coordinates
  — use SMOPCA/SpatialPCA instead, since plain PCA assumes independence among cells
  and ignores spatial correlation
- Peak-level ATAC analysis where the count matrix is sparse and peak-based feature
  selection (e.g. peaks detected in ≥200 cells via FindTopFeatures) has not yet been
  performed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Seurat V4
  role: Runs feature selection (FindVariableFeatures), scaling, and PCA on the normalized
    single-cell multi-omics matrix; its RunPCA output feeds UMAP to generate pseudo-spatial
    coordinates
  repo: https://github.com/satijalab/seurat
- name: Signac
  role: Provides FindTopFeatures to select peaks detected in at least 200 cells before
    PCA on ATAC-derived gene activity matrices
- name: Specter
  role: Consumes PCA-reduced representations for downstream multi-modal clustering
    of CITE-seq RNA and ADT data
  repo: https://github.com/canzarlab/Specter
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:cb7aa3e95a375fb7099c0f1ea4adc4fcf626e5e28dc773b86a2202a5bb0efd41
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
  - Signac
  - Specter
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# pca-dimension-reduction-single-cell

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Dimension reduction of single-cell multi-omics count matrices via PCA, as used in SMOPCA: features are selected (top 2000 highly variable genes for CITE-seq or mapped genes for SMAGE-seq), normalized by library size with log transformation and unit-variance scaling, and projected onto principal components in Seurat V4 to yield low-dimensional cell representations.

## When to use

Use when a single-cell CITE-seq or SMAGE-seq count matrix needs a linear low-dimensional embedding before UMAP visualization, clustering, or as SMOPCA's downstream input.

## When NOT to use

- Input is already a low-dimensional embedding or PC matrix — re-running PCA adds no value
- The goal is spatially aware dimension reduction on data with real spatial coordinates — use SMOPCA/SpatialPCA instead, since plain PCA assumes independence among cells and ignores spatial correlation
- Peak-level ATAC analysis where the count matrix is sparse and peak-based feature selection (e.g. peaks detected in ≥200 cells via FindTopFeatures) has not yet been performed

## Inputs

- Preprocessed single-cell multi-omics count matrix (e.g. CITE-seq RNA counts with ADT, or SMAGE-seq gene activity matrix)
- List of selected features: top 2000 highly variable genes (CITE-seq) or mapped genes (SMAGE-seq)

## Outputs

- Matrix of cell scores on retained principal components (top 50 PCs for UMAP input)
- Low-dimensional representation used downstream for UMAP coordinates, K-means clustering labels, or SMOPCA latent factor estimation

## How to apply

Load the preprocessed count matrix and normalize by library size (total read count), then log-transform and scale to zero mean and unit variance. Filter out genes and proteins with zero counts, then select the top 2000 highly variable genes (CITE-seq data) or mapped genes (SMAGE-seq data, e.g. ATAC peaks collapsed into a gene activity matrix) as input features. Run PCA on the selected features in Seurat V4 and retain the top 50 principal components when the embedding will feed UMAP; 20 PCs are adequate for t-SNE visualization and graph-based clustering per the Specter multimodal tutorial. Rationale: PCA on HVGs concentrates the variance signal so the downstream latent factors (e.g. SMOPCA's posterior mean of Z) are computed on a denoised, low-rank feature space.

## Related tools

- **Seurat V4** (Runs feature selection (FindVariableFeatures), scaling, and PCA on the normalized single-cell multi-omics matrix; its RunPCA output feeds UMAP to generate pseudo-spatial coordinates) — https://github.com/satijalab/seurat
- **Signac** (Provides FindTopFeatures to select peaks detected in at least 200 cells before PCA on ATAC-derived gene activity matrices)
- **Specter** (Consumes PCA-reduced representations for downstream multi-modal clustering of CITE-seq RNA and ADT data) — https://github.com/canzarlab/Specter

## Examples

```
pbmc <- NormalizeData(pbmc); pbmc <- FindVariableFeatures(pbmc); pbmc <- ScaleData(pbmc); pbmc <- RunPCA(pbmc, verbose = FALSE)
```

## Evaluation signals

- Zero-count genes/proteins are removed before feature selection
- Exactly 2000 highly variable genes (CITE-seq) or mapped genes (SMAGE-seq) are passed to RunPCA
- The retained PC set matches the downstream need (top 50 PCs for UMAP; 20 PCs for t-SNE/graph clustering)
- Scaled data has zero mean and unit variance per feature after normalization by library size and log transformation
- Downstream UMAP computed on the PCs yields a 2D coordinate per cell, e.g. written to umap_coordinates.csv

## Limitations

- PCA-based pipelines operate on normalized data rather than raw counts, which the SMOPCA authors note can cause a loss of inference accuracy and reduced analytical power
- Linear PCA does not model dropout events or the mean–variance relationship of raw counts, unlike count-aware deep generative alternatives such as scVI/totalVI
- Plain PCA assumes independence among cells and fails to model spatial dependencies, so it is suboptimal for spatial multi-omics data without a spatially aware wrapper

## Evidence

- [discussion] Feature selection thresholds: [evidence span withheld — anchor status: not_found]
