---
name: paired-transcriptome-proteome-data-preprocessing
description: Use when you have paired RNA + protein measurements in anndata/h5ad format
  (e.g., CITE-seq or BMNC/SLN111_D1 style data) that must be normalized into consistent
  formats, dimensions, units, and normalizations before integration with methods like
  SSGATE.
when_to_use_negative:
- Input is spatial multi-omics data where spatial coordinates must be preserved —
  ensure coordinates are kept in anndata.obsm['spatial'] rather than treating the
  data as generic expression matrices
- Integrating omics types beyond transcriptome/proteome — the article notes that normalization
  and dimensionality reduction methods for other single-omics data are not yet available
- Data already normalized to a common scale or already feature-reduced, where re-scaling
  and log1p could distort the distributions
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: SCANPY
  role: 'Used to preprocess the BMNC and SLN111_D1 paired datasets: count-depth scaling
    with log1p transformation plus highly variable gene selection for RNA, and centered
    log-ratio transformation for protein'
- name: SSGATE
  role: Downstream consumer of the preprocessed AnnData objects; requires data in
    anndata format with expression profiles in anndata.X
  repo: https://github.com/Linliu-Bioinf/SSGATE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:ef7175651f1ecd42830235528d321ed144374f773d987d6e78378a02d3eace1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Linliu-Bioinf/SSGATE
  tools:
  - MOFA
  - SCANPY
  - SSGATE
  - ScMM
  - Seurat v5
  - TotalVI
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# paired-transcriptome-proteome-data-preprocessing

## Summary

Preprocesses paired transcriptome–proteome single-cell datasets (stored as h5ad/AnnData) so both modalities are comparable before multi-omics integration: RNA is count-depth scaled with log1p transformation plus highly variable gene selection, while protein abundances receive centered log-ratio (CLR) transformation, as done for the BMNC and SLN111_D1 benchmarks in SSGATE.

## When to use

Use when you have paired RNA + protein measurements in anndata/h5ad format (e.g., CITE-seq or BMNC/SLN111_D1 style data) that must be normalized into consistent formats, dimensions, units, and normalizations before integration with methods like SSGATE.

## When NOT to use

- Input is spatial multi-omics data where spatial coordinates must be preserved — ensure coordinates are kept in anndata.obsm['spatial'] rather than treating the data as generic expression matrices
- Integrating omics types beyond transcriptome/proteome — the article notes that normalization and dimensionality reduction methods for other single-omics data are not yet available
- Data already normalized to a common scale or already feature-reduced, where re-scaling and log1p could distort the distributions

## Inputs

- Paired transcriptome–proteome h5ad files (e.g., BMNC, SLN111_D1 from the BGI cloud disk linked in the SSGATE repository)
- AnnData objects with expression profiles stored in anndata.X

## Outputs

- Preprocessed AnnData objects: RNA normalized via count-depth scaling + log1p with HVG selection; protein CLR-transformed
- Modality-specific matrices ready as inputs to SSGATE's Cal_Nbrs_Net / train functions

## How to apply

Load the paired transcriptome–proteome h5ad files and keep expression profiles in anndata.X so downstream tools can read them correctly. In SCANPY, apply count-depth scaling with log1p transformation to the RNA modality and perform highly variable gene selection to reduce dimensionality and noise. Transform the protein modality with centered log-ratio (CLR) normalization, which addresses compositional effects in antibody-derived tags. Keep the two modalities in the same AnnData object structure (e.g., adata_st for transcriptomics, adata_sp for proteomics when passing to SSGATE) so formats, dimensions, units, and normalizations are consistent; this consistency is the rationale given in the article for careful cross-omics preprocessing.

## Related tools

- **SCANPY** (Used to preprocess the BMNC and SLN111_D1 paired datasets: count-depth scaling with log1p transformation plus highly variable gene selection for RNA, and centered log-ratio transformation for protein)
- **SSGATE** (Downstream consumer of the preprocessed AnnData objects; requires data in anndata format with expression profiles in anndata.X) — https://github.com/Linliu-Bioinf/SSGATE

## Examples

```
import scanpy as sc; adata_rna = sc.read_h5ad('BMNC_rna.h5ad'); sc.pp.normalize_total(adata_rna); sc.pp.log1p(adata_rna); sc.pp.highly_variable_genes(adata_rna); adata_prot = sc.read_h5ad('BMNC_prot.h5ad'); adata_prot.X = sc.pp.normalize_total(adata_prot, target_sum=None, exclude_highly_expressed=True, inplace=False)['X']
```

## Evaluation signals

- RNA modality is log1p-transformed after count-depth scaling and restricted to highly variable genes; check adata.X distributions and var_genes flags
- Protein modality is CLR-transformed (per-cell centered log-ratio); verify each cell's protein vector sums to zero in log space
- Expression profiles are stored in anndata.X (not layers/obsm) so SSGATE can read them; a read failure here indicates misformatting
- Both modalities retain the same number of cells (paired) with matching obs indices
- Downstream SSGATE benchmark validation: with this preprocessing, SSGATE attains average ranking 1.5 on BMNC and 2.25 (tied with totalVI) on SLN111_D1 with smallest RRA p-value

## Limitations

- Normalization and dimensionality reduction methods for integrating omics beyond transcriptome/proteome are not yet available
- If spatial coordinates are not stored in anndata.obsm['spatial'], spatial multi-omics data will be processed as single-cell multi-omics data because the spatial information cannot be read
- Datasets used in the study are provided via a BGI cloud disk link with an extract password, which may limit programmatic reproducibility

## Evidence

- [other] preprocess them in SCANPY: count-depth scaling with log1p transformation plus highly variable gene selection for RNA, and centered log-ratio transformation for protein: [evidence span withheld — anchor status: not_found]
- [readme] SSGATE requires data in anndata format as inputs; expression profiles must be in anndata.X: "SSGATE requires data in [anndata](https://anndata.readthedocs.io/en/latest/) format as inputs. 
The expression profiles are required to be stored in anndata.X, otherwise the data cannot be read"
- [intro] Formats, dimensions, units, and normalizations of single-omics data can differ significantly and require consistency efforts: "Sometimes these differences can be significant, requiring efforts to ensure data consistency in terms of formats, dimensions, units, and normalizations [12]."
- [discussion] Normalization for other single-omics data not yet available: "the integration of other single-omics data requires reasonable normalization and dimensionality reduction methods, which is also the focus of our subsequent research."
