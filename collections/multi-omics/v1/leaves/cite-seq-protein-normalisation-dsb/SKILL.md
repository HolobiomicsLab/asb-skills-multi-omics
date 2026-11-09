---
name: cite-seq-protein-normalisation-dsb
description: Use when raw 'Antibody capture' counts from a 10x CITE-seq run must be
  background-corrected before annotation or multimodal integration, and both the filtered
  and raw Cell Ranger matrices are available.
when_to_use_negative:
- No raw/unfiltered matrix with empty droplets is available — dsb requires the background
  droplets to estimate background signal; consider CLR normalisation (pt.pp.clr) instead
- The data is single-modality RNA-seq or scATAC-seq with no 'Antibody capture' features
  — this skill targets CITE-seq-style protein counts only
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: muon
  role: Reads filtered and raw 10x HDF5 matrices (muon.read_10x_h5) and hosts the
    pt.pp.dsb normalisation for the protein modality
  repo: github.com/scverse/muon
- name: dsb
  role: Original R implementation of denoising-and-scaling normalisation of antibody-derived-tag
    data that MUON wraps in pt.pp.dsb
  repo: github.com/niaid/dsb
- name: SCANPY
  role: Single-cell gene expression analysis used alongside MUON for downstream handling
    of the normalised modalities
- name: NumPy
  role: Array computations underpinning the normalised count matrices
- name: pandas
  role: Data-frame handling for feature metadata during normalisation
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:7e12cf2e529f3fefdd134bd8dff1b06893d2194deebd0b63c1a80e854f2fbccb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MUON
  - NumPy
  - SCANPY
  - dsb
  - muon
  - pandas
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cite-seq-protein-normalisation-dsb

## Summary

Normalises CITE-seq antibody-derived-tag (protein) counts with the dsb denoising-and-scaling method, using low-RNA-content empty droplets as a background reference so protein abundance can be used for cell-type definition akin to flow cytometry gating.

## When to use

Use when raw 'Antibody capture' counts from a 10x CITE-seq run must be background-corrected before annotation or multimodal integration, and both the filtered and raw Cell Ranger matrices are available.

## When NOT to use

- No raw/unfiltered matrix with empty droplets is available — dsb requires the background droplets to estimate background signal; consider CLR normalisation (pt.pp.clr) instead
- The data is single-modality RNA-seq or scATAC-seq with no 'Antibody capture' features — this skill targets CITE-seq-style protein counts only

## Inputs

- Filtered Cell Ranger feature-barcode HDF5 matrix (10x CITE-seq) containing 'rna' and 'prot' modalities
- Raw (unfiltered) Cell Ranger feature-barcode HDF5 matrix used to define background/empty droplets
- empty_counts_range parameter selecting empty droplets by low RNA counts

## Outputs

- Denoised and scaled protein abundance AnnData/MuData modality with finite, background-corrected values per antibody feature
- An .h5mu file storing the normalised protein modality (e.g. mdata_integrated.h5mu with the .mod group)
- Per-feature distribution plots of normalised protein abundance for all measured epitopes (Fig. 3c)

## How to apply

Read the filtered and raw Cell Ranger HDF5 feature-barcode matrices with muon.read_10x_h5 so both the cell-associated counts and the raw matrix with background droplets are loaded. Extract the protein ('Antibody capture') modality from each MuData object, then run pt.pp.dsb(adata_prot, adata_prot_raw, empty_counts_range=...), where empty_counts_range selects empty droplets by low RNA counts to estimate and remove background protein signal. The result is a denoised and scaled protein matrix whose values can be interpreted like flow-cytometry gates for defining cell types, and visualised as per-feature distributions (Fig. 3c). The normalised modality can be written to an .h5mu file or intersected with the RNA modality via mu.pp.intersect_obs before joint integration (e.g. MOFA+ or WNN, Fig. 3d).

## Related tools

- **muon** (Reads filtered and raw 10x HDF5 matrices (muon.read_10x_h5) and hosts the pt.pp.dsb normalisation for the protein modality) — github.com/scverse/muon
- **dsb** (Original R implementation of denoising-and-scaling normalisation of antibody-derived-tag data that MUON wraps in pt.pp.dsb) — github.com/niaid/dsb
- **SCANPY** (Single-cell gene expression analysis used alongside MUON for downstream handling of the normalised modalities)
- **NumPy** (Array computations underpinning the normalised count matrices)
- **pandas** (Data-frame handling for feature metadata during normalisation)

## Examples

```
mdata = mu.read_10x_h5('filtered_feature_bc_matrix.h5'); mdata_raw = mu.read_10x_h5('raw_feature_bc_matrix.h5'); mu.prot.pp.dsb(mdata['prot'], mdata_raw['prot'], empty_counts_range=(1,10))
```

## Evaluation signals

- The output .h5mu file exists and contains the .mod group with the 'prot' (and 'rna') modalities
- The normalised protein matrix contains finite (non-NaN, non-inf) background-corrected values for every antibody feature
- Per-feature distributions of the normalised protein values are well separated into background-negative and positive peaks (Fig. 3c style), supporting gating-like interpretation
- After mu.pp.intersect_obs, observations are shared across both modalities and downstream checks such as 'X_mofa' in mdata.obsm evaluate True

## Limitations

- dsb depends on the raw matrix containing sufficiently many empty droplets, selected via empty_counts_range; if the raw matrix or low-RNA background population is missing, the background estimate is unreliable
- Support for emerging/other multi-omics assay types beyond those MUON currently handles is listed as future work, so dsb in MUON applies to CITE-seq-like ADT assays rather than arbitrary modalities

## Evidence

- [methods] dsb normalisation with background droplets: "pt.pp.dsb(adata_prot, adata_prot_raw, empty_counts_range=...)"
- [methods] intersect observations across modalities: "In order to keep observations present in all the modalities, there is {func}`muon.pp.intersect_obs`."
- [readme] dsb package purpose: [evidence span withheld — anchor status: external_doc]
- [abstract] MUON framework scope: "we present a data standard and an analysis framework for multi-omics, MUON, designed to organise, analyse, visualise, and exchange multimodal data"
