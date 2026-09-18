---
name: single-cell-multimodal-matrix-io
description: Use when input data are 10x Cell Ranger HDF5 (feature-barcode) matrices
  that must be loaded as multimodal MuData objects, split into modalities such as
  'Antibody capture', and serialised to .h5mu for downstream processing.
when_to_use_negative:
- Input is already a normalised AnnData/.h5mu object and no re-import from Cell Ranger
  output is needed
- Data are 10x scATAC fragment/snAP files rather than HDF5 feature-barcode matrices
  (use muon.atac read_snap-style loading instead)
- Only a single-omics (RNA-only) dataset with no multimodal structure is analysed
  — a plain AnnData/SCANPY workflow suffices
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: MUON
  role: Provides muon.read_10x_h5 for loading 10x HDF5 matrices into MuData, modality
    extraction, and .h5mu serialisation
  repo: https://github.com/scverse/muon
- name: SCANPY
  role: Underlying single-cell analysis library reused within muon for per-modality
    processing
- name: NumPy
  role: Array operations for checking finite values in the normalised protein matrix
- name: pandas
  role: Table handling for feature/observation metadata accompanying the matrices
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:b1cdeba17d48e6141d637a10cb64b3f4363d7ff4c33b7beeb565a0000cec01cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/muon
  tools:
  - MUON
  - NumPy
  - SCANPY
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# single-cell-multimodal-matrix-io

## Summary

Reads filtered and raw 10x Cell Ranger HDF5 feature-barcode matrices into MuData objects via muon.read_10x_h5 and writes normalised multimodal modalities to .h5mu files, giving interoperable multimodal I/O for workflows such as CITE-seq protein denoising.

## When to use

Use when input data are 10x Cell Ranger HDF5 (feature-barcode) matrices that must be loaded as multimodal MuData objects, split into modalities such as 'Antibody capture', and serialised to .h5mu for downstream processing.

## When NOT to use

- Input is already a normalised AnnData/.h5mu object and no re-import from Cell Ranger output is needed
- Data are 10x scATAC fragment/snAP files rather than HDF5 feature-barcode matrices (use muon.atac read_snap-style loading instead)
- Only a single-omics (RNA-only) dataset with no multimodal structure is analysed — a plain AnnData/SCANPY workflow suffices

## Inputs

- Filtered 10x Cell Ranger HDF5 feature-barcode matrix (e.g. filtered_feature_bc_matrix.h5)
- Raw 10x Cell Ranger HDF5 matrix (raw_feature_bc_matrix.h5) used as background/empty-droplet reference
- Optionally 10x MTX directories with matrix.mtx, features.tsv.gz, barcodes.tsv.gz via muon.read_10x_mtx

## Outputs

- MuData objects containing gene-expression and protein ('Antibody capture') modalities
- Normalised .h5mu file serialising the multimodal object
- Protein AnnData modality with background-corrected dsb-normalised values

## How to apply

Load both the filtered cell-associated matrix and the raw matrix with muon.read_10x_h5; the raw matrix is required to define background droplets for dsb normalisation. Extract the 'Antibody capture' (protein) AnnData modality from each MuData object using dictionary-style modality access (e.g. mdata['prot']). After downstream normalisation (e.g. pt.pp.dsb with empty_counts_range selecting background droplets by low RNA counts), write the MuData object to .h5mu via MuData.write. HDF5 serialisation keeps MuData objects accessible from other languages such as R and Julia, so the output is a portable multimodal exchange format. Validate by confirming the .h5mu file exists and the modality matrix contains finite values for every antibody feature.

## Related tools

- **MUON** (Provides muon.read_10x_h5 for loading 10x HDF5 matrices into MuData, modality extraction, and .h5mu serialisation) — https://github.com/scverse/muon
- **SCANPY** (Underlying single-cell analysis library reused within muon for per-modality processing)
- **NumPy** (Array operations for checking finite values in the normalised protein matrix)
- **pandas** (Table handling for feature/observation metadata accompanying the matrices)

## Examples

```
import muon as mu; mdata = mu.read_10x_h5('filtered_feature_bc_matrix.h5'); mdata_raw = mu.read_10x_h5('raw_feature_bc_matrix.h5'); prot = mdata['prot']; mdata.write('pbmc_cite_seq.h5mu')
```

## Evaluation signals

- The output .h5mu file exists at the expected path and can be re-opened with muon.read
- The 'Antibody capture' modality is present in the MuData object and aligned on a shared set of observations with other modalities
- Every antibody feature in the normalised protein matrix has finite (non-NaN, non-inf) background-corrected values
- Both filtered and raw matrices were loaded; the raw matrix background droplets are identified via empty_counts_range on low RNA counts

## Limitations

- The raw (unfiltered) matrix must be available; without it, dsb background estimation from empty droplets cannot be performed
- empty_counts_range thresholds must be set per-dataset to select background droplets by low RNA counts; wrong ranges corrupt background correction
- Handling of novel multi-omics assays (e.g. genomic-coordinate or spatially resolved assays) is future work in muon rather than a current capability

## Evidence

- [methods] read_10x_h5 for loading filtered and raw Cell Ranger HDF5 matrices: "with {func}`muon.read_10x_h5`"
- [methods] dsb protein normalisation call signature: "pt.pp.dsb(adata_prot, adata_prot_raw, empty_counts_range=...)"
- [discussion] HDF5 serialisation enables cross-language access to MuData objects: "Serialisation to HDF5 makes MuData objects accessible to other programming languages, including R and Julia."
- [readme] muon is a multimodal omics framework: "`muon` is a multimodal omics Python framework."
- [readme] muon-tutorials CITE-seq tutorial covering ADT count normalisation: [evidence span withheld — anchor status: not_found]
