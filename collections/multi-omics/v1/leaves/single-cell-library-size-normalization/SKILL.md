---
name: single-cell-library-size-normalization
description: Use when the input is a raw count matrix from CITE-seq, SMAGE-seq, or
  spatial multi-omics data that must be normalized and scaled before downstream feature
  selection, PCA, UMAP, or spatial dimension reduction.
when_to_use_negative:
- Input is already a normalized/scaled expression matrix or a dimension-reduced representation
  (e.g., PCs or UMAP coordinates), not raw counts
- The downstream method expects raw count data, e.g., count-aware deep generative
  models like TotalVI or scVI that model the mean-variance relationship of raw counts
- Data are non-count modalities such as already CLR-normalized ADT matrices, where
  per-library-size normalization of RNA-style counts is inappropriate
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat V4
  role: R toolkit used to normalize by library size, log-transform, scale to zero
    mean/unit variance, select top 2000 highly variable genes, and run PCA/UMAP on
    the normalized features
  repo: https://github.com/satijalab/seurat
- name: BREM-SC
  role: Benchmark single-cell multi-omics clustering method whose raw-count input
    contrasts with the normalized-data pipeline of this skill
  repo: https://github.com/tarot0410/BREMSC
- name: Specter
  role: Downstream multimodal clustering tool whose README demonstrates Seurat NormalizeData/ScaleData
    preprocessing before clustering
  repo: https://github.com/canzarlab/Specter
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:9fbfc1a0bb1326e874ce39ed1b5ede8f23966927f4440607c62adc98612b1286
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
  - Specter
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# single-cell-library-size-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalizes single-cell or spatial multi-omics count matrices by library size (total read count), applies log transformation, and scales features to zero mean and unit variance, producing the preprocessed input required before feature selection, PCA/UMAP, or SMOPCA dimension reduction.

## When to use

Use when the input is a raw count matrix from CITE-seq, SMAGE-seq, or spatial multi-omics data that must be normalized and scaled before downstream feature selection, PCA, UMAP, or spatial dimension reduction.

## When NOT to use

- Input is already a normalized/scaled expression matrix or a dimension-reduced representation (e.g., PCs or UMAP coordinates), not raw counts
- The downstream method expects raw count data, e.g., count-aware deep generative models like TotalVI or scVI that model the mean-variance relationship of raw counts
- Data are non-count modalities such as already CLR-normalized ADT matrices, where per-library-size normalization of RNA-style counts is inappropriate

## Inputs

- Raw single-cell multi-omics count matrix (CITE-seq RNA + ADT, e.g., PBMC 3762 cells x 33,538 genes x 49 proteins)
- SMAGE-seq mapped gene activity count matrix (PBMC3K ~3000 cells; PBMC10K ~10000 cells)
- Spatial multi-omics count matrices (e.g., spatial-CITE-seq human tonsil GSE213264, Stereo-CITE-seq mouse thymus)

## Outputs

- Library-size-normalized, log-transformed, scaled gene/protein expression matrix (zero mean, unit variance)
- Normalized feature matrix restricted to top 2000 highly variable genes (CITE-seq) or mapped genes (SMAGE-seq), ready for Seurat PCA and UMAP

## How to apply

Start from the raw gene/protein count matrix (e.g., CITE-seq RNA with 33,538 genes and 49 proteins in the PBMC dataset, or SMAGE-seq mapped gene matrices) and first filter out genes and proteins with zero counts. Normalize the count matrix by library size (the total read count), then apply a log transformation, and finally scale to zero mean and unit variance. On this normalized matrix, select the top 2000 highly variable genes (CITE-seq data) or mapped genes (SMAGE-seq data) as input features using Seurat, then run PCA and, for pseudo-spatial workflows, compute the 2D UMAP embedding from the top 50 PCs. Rationale: SMOPCA and its UMAP-based pseudo-spatial coordinate generation consume normalized, scaled matrices; skipping this step changes downstream PCA/UMAP geometry and the resulting location input.

## Related tools

- **Seurat V4** (R toolkit used to normalize by library size, log-transform, scale to zero mean/unit variance, select top 2000 highly variable genes, and run PCA/UMAP on the normalized features) — https://github.com/satijalab/seurat
- **BREM-SC** (Benchmark single-cell multi-omics clustering method whose raw-count input contrasts with the normalized-data pipeline of this skill) — https://github.com/tarot0410/BREMSC
- **Specter** (Downstream multimodal clustering tool whose README demonstrates Seurat NormalizeData/ScaleData preprocessing before clustering) — https://github.com/canzarlab/Specter

## Examples

```
pbmc <- CreateSeuratObject(counts = pbmc.rna); pbmc <- NormalizeData(pbmc); pbmc <- FindVariableFeatures(pbmc); pbmc <- ScaleData(pbmc)
```

## Evaluation signals

- Post-normalization, each cell's total normalized counts (library size) should be comparable across cells in the same modality
- After scaling, selected gene features should have approximately zero mean and unit variance across cells
- Genes and proteins with zero counts must be absent from the normalized matrix (they were filtered out initially)
- Exactly the top 2000 features (HVGs for CITE-seq; mapped genes for SMAGE-seq) should survive into the PCA/UMAP input
- Downstream PCA on the normalized matrix should yield the top 50 PCs used for UMAP without errors or NaNs

## Limitations

- SMOPCA-style downstream analysis operates on normalized data only, not raw counts, potentially causing loss of inference accuracy and reduced analytical power since dropout events and the mean-variance relationship of raw counts are not modeled
- Library-size normalization does not address modality-specific normalization needs, e.g., ADT/protein data typically require centered log-ratio (CLR) normalization as shown in the Seurat/Specter workflows

## Evidence

- [discussion] Top 2000 feature selection on the normalized matrix: [evidence span withheld — anchor status: not_found]
