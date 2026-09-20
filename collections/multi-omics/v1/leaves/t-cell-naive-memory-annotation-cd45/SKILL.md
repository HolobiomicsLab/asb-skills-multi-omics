---
name: t-cell-naive-memory-annotation-cd45
description: Use when annotating T cell states in CITE-seq PBMC data where the naive-vs-memory
  distinction needs to be attributed to protein-level CD45 RA/RO abundance rather
  than gene expression alone.
when_to_use_negative:
- Input lacks a protein/ADT modality (e.g. gene-expression-only scRNA-seq) — CD45
  RA/RO attribution requires epitope abundance data
- Raw unfiltered matrix with empty droplets is unavailable, so the dsb denoised-and-scaled-by-background
  normalisation cannot be applied
- Goal is single-modality clustering without multimodal integration — a plain Scanpy
  workflow on the 'rna' modality alone suffices
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: muon
  role: Reads 10x H5 matrices into 'rna'/'prot' modalities, intersects observations,
    runs MOFA+ integration, builds WNN neighbours and UMAP, writes .h5mu
  repo: github.com/scverse/muon
- name: dsb
  role: Denoised-and-scaled-by-background normalisation of protein counts using empty
    droplets (pt.pp.dsb)
  repo: github.com/niaid/dsb
- name: MOFA+
  role: Unsupervised joint factor analysis of both modalities (mu.tl.mofa) storing
    factors in X_mofa
- name: scanpy
  role: Single-cell analysis utilities reused within the muon workflow
- name: muon-tutorials
  role: CITE-seq PBMC 5k tutorial notebook demonstrating processing and integration
    of surface protein abundance and gene expression
  repo: github.com/scverse/muon-tutorials
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:73322655abb96c7d9d38e5e43e9869c046df03f92caeb4b0300e45cfd7f70500
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MOFA+
  - NumPy
  - dsb
  - muon
  - muon-tutorials
  - pandas
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# t-cell-naive-memory-annotation-cd45

## Summary

Annotates naive vs memory T cells in PBMC CITE-seq data by jointly integrating gene expression and protein modalities with MUON (MOFA+ factors + WNN neighbours/UMAP) and attributing the naive-memory split to CD45 RA/RO isoform abundance at the protein level, reproducing the Hao et al. 2020 WNN analysis.

## When to use

Use when annotating T cell states in CITE-seq PBMC data where the naive-vs-memory distinction needs to be attributed to protein-level CD45 RA/RO abundance rather than gene expression alone.

## When NOT to use

- Input lacks a protein/ADT modality (e.g. gene-expression-only scRNA-seq) — CD45 RA/RO attribution requires epitope abundance data
- Raw unfiltered matrix with empty droplets is unavailable, so the dsb denoised-and-scaled-by-background normalisation cannot be applied
- Goal is single-modality clustering without multimodal integration — a plain Scanpy workflow on the 'rna' modality alone suffices

## Inputs

- Filtered Cell Ranger feature-barcode matrix (H5) with 'rna' and 'prot' modalities, readable by muon.read_10x_h5
- Raw (unfiltered) Cell Ranger feature-barcode matrix containing background/empty droplets for dsb background estimation

## Outputs

- MuData object with joint factors in mdata.obsm['X_mofa'] and UMAP coordinates in mdata.obsm
- UMAP embedding with naive vs memory T cell annotations and CD45 RA/RO protein abundance visualisation
- Saved .h5mu file (mdata_integrated.h5mu) containing the .mod group with both 'rna' and 'prot' modalities

## How to apply

Load the filtered and raw Cell Ranger feature-barcode matrices with muon.read_10x_h5 to obtain 'rna' and 'prot' modalities plus background droplets. Normalise protein counts with pt.pp.dsb(adata_prot, adata_prot_raw, empty_counts_range=...), using low-RNA-content empty droplets to estimate and remove background ADT signal. Intersect observations across modalities with mu.pp.intersect_obs(mdata), then run mu.tl.mofa(mdata) with default feature selection use_var='highly_variable' to learn a joint latent space stored in mdata.obsm['X_mofa']. Build the multimodal weighted-nearest-neighbours graph with mu.pp.neighbors and embed with mu.tl.umap; the rationale is that combining both modalities lets the naive-memory split be attributed to CD45 RA/RO protein abundance, visualised on the UMAP. Save with mdata.write('mdata_integrated.h5mu').

## Related tools

- **muon** (Reads 10x H5 matrices into 'rna'/'prot' modalities, intersects observations, runs MOFA+ integration, builds WNN neighbours and UMAP, writes .h5mu) — github.com/scverse/muon
- **dsb** (Denoised-and-scaled-by-background normalisation of protein counts using empty droplets (pt.pp.dsb)) — github.com/niaid/dsb
- **MOFA+** (Unsupervised joint factor analysis of both modalities (mu.tl.mofa) storing factors in X_mofa)
- **scanpy** (Single-cell analysis utilities reused within the muon workflow)
- **muon-tutorials** (CITE-seq PBMC 5k tutorial notebook demonstrating processing and integration of surface protein abundance and gene expression) — github.com/scverse/muon-tutorials

## Examples

```
import muon as mu; mdata = mu.read_10x_h5('filtered_feature_bc_matrix.h5'); mu.pp.intersect_obs(mdata); mu.tl.mofa(mdata, use_var='highly_variable'); mu.pp.neighbors(mdata); mu.tl.umap(mdata); mdata.write('mdata_integrated.h5mu')
```

## Evaluation signals

- "X_mofa" in mdata.obsm evaluates True after mu.tl.mofa
- UMAP coordinates are present in mdata.obsm after mu.tl.umap
- The saved .h5mu file contains the .mod group with both the 'rna' and 'prot' modalities
- Naive and memory T cells separate on the WNN/UMAP embedding and the split aligns with CD45 RA/RO protein abundance rather than RNA alone
- mu.pp.intersect_obs leaves only observations present in both modalities

## Limitations

- Support for emerging multi-omics assays is identified as future work for MUON rather than a current capability
- The naive-memory attribution depends on dsb background estimation from empty droplets; poor empty-droplet coverage (empty_counts_range mis-specified) degrades protein normalisation
- Single-omics methods can only be reused in multi-omics workflows because MUON is modular; relying on gene expression alone cannot attribute the split to CD45 isoforms

## Evidence

- [methods] dsb normalisation step: [evidence span withheld — anchor status: not_found]
- [methods] intersect observations: "In order to keep observations present in all the modalities, there is {func}`muon.pp.intersect_obs`."
- [abstract] muon purpose: "we present a data standard and an analysis framework for multi-omics, MUON, designed to organise, analyse, visualise, and exchange multimodal data"
- [readme] muon-tutorials CITE-seq tutorial: [evidence span withheld — anchor status: not_found]
- [discussion] modular reuse of single-omics tools: "MUON is designed in a modular fashion, which means that existing methods and tools for processing individual omics can be reused to design more complex analysis workflows"
