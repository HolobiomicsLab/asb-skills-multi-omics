---
name: dsb-background-denoising
description: Use when CITE-seq 'Antibody capture' ADT counts need background-corrected
  normalisation before cell-type definition or gating-like downstream analysis, and
  both filtered and raw feature-barcode matrices are available.
when_to_use_negative:
- Only the filtered matrix is available and no raw matrix exists to supply the empty/background
  droplet reference required by dsb.
- The dataset has no 'Antibody capture' modality (e.g. pure gene-expression or ATAC-seq
  data, where muon offers tfidf/binarize instead).
- ADT counts are already normalised by another scheme such as clr (pt.pp.clr(mdata['prot'])),
  so applying dsb again would double-normalise.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MUON
  role: reads 10x HDF5 matrices, holds the 'Antibody capture' AnnData modality, and
    exposes pt.pp.dsb for dsb normalisation on protein counts
  repo: https://github.com/scverse/muon
- name: dsb
  role: original R implementation of the denoising-and-scaling algorithm that muon's
    pt.pp.dsb wraps
  repo: https://github.com/niaid/dsb
- name: SCANPY
  role: single-cell preprocessing/plotting utilities used alongside muon on the normalised
    protein data
- name: NumPy
  role: array handling for finite-value checks on the normalised protein matrix
- name: pandas
  role: data frame handling for per-feature protein abundance summaries
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:e12c2b5601ca519f761ec8d8a6163f1e667c14d1a9672a951fee3ac5468c85bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/muon
  tools:
  - MUON
  - NumPy
  - SCANPY
  - dsb
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# dsb-background-denoising

## Summary

Applies dsb denoising-and-scaling normalisation to CITE-seq antibody-derived-tag (ADT) protein counts via muon's pt.pp.dsb, using the raw Cell Ranger matrix as a background reference so normalised protein abundance can define cell types akin to flow-cytometry gating (Fig. 3c).

## When to use

Use when CITE-seq 'Antibody capture' ADT counts need background-corrected normalisation before cell-type definition or gating-like downstream analysis, and both filtered and raw feature-barcode matrices are available.

## When NOT to use

- Only the filtered matrix is available and no raw matrix exists to supply the empty/background droplet reference required by dsb.
- The dataset has no 'Antibody capture' modality (e.g. pure gene-expression or ATAC-seq data, where muon offers tfidf/binarize instead).
- ADT counts are already normalised by another scheme such as clr (pt.pp.clr(mdata['prot'])), so applying dsb again would double-normalise.

## Inputs

- Filtered Cell Ranger HDF5 feature-barcode matrix (10x h5, e.g. via muon.read_10x_h5)
- Raw Cell Ranger HDF5 feature-barcode matrix (used to define background/empty droplets)
- Protein 'Antibody capture' modality extracted as an AnnData object from the MuData container

## Outputs

- Normalised protein AnnData modality with denoised, background-corrected ADT values
- .h5mu file containing the normalised protein modality
- Per-feature distribution plots of normalised protein abundance for all measured epitopes (Fig. 3c style)

## How to apply

Read both the filtered and raw Cell Ranger HDF5 feature-barcode matrices with muon.read_10x_h5, because dsb needs the raw matrix to define empty/background droplets. Extract the 'Antibody capture' (protein) AnnData modality from each MuData object. Run pt.pp.dsb on the protein modality, passing the raw protein matrix as the empty/background reference and setting empty_counts_range to select background droplets by low RNA counts; this estimates and removes background protein signal. Write the normalised protein modality to an .h5mu file, then plot per-feature distributions of the normalised protein abundances for all measured epitopes (Fig. 3c). dsb is muon's Python interface to the approach from Mulè, Martins, and Tsang 2022, whose original R implementation lives at github.com/niaid/dsb.

## Related tools

- **MUON** (reads 10x HDF5 matrices, holds the 'Antibody capture' AnnData modality, and exposes pt.pp.dsb for dsb normalisation on protein counts) — https://github.com/scverse/muon
- **dsb** (original R implementation of the denoising-and-scaling algorithm that muon's pt.pp.dsb wraps) — https://github.com/niaid/dsb
- **SCANPY** (single-cell preprocessing/plotting utilities used alongside muon on the normalised protein data)
- **NumPy** (array handling for finite-value checks on the normalised protein matrix)
- **pandas** (data frame handling for per-feature protein abundance summaries)

## Examples

```
mdata = mu.read_10x_h5('filtered.h5'); prot = mdata['prot']; pt.pp.dsb(prot, mu.read_10x_h5('raw.h5')['prot'], empty_counts_range=(1, 50))
```

## Evaluation signals

- The output .h5mu file exists and can be reloaded.
- The normalised protein matrix contains finite (non-NaN, non-inf) values for every antibody feature.
- Values are background-corrected, not raw counts (raw ADT counts are non-negative integers; denoised values may go negative).
- All antibody-capture features are retained after normalisation (no features dropped by the dsb step).
- Per-feature distributions support cell-type separation comparable to flow-cytometry gating (Fig. 3c).

## Limitations

- dsb requires a raw matrix with empty/background droplets; empty_counts_range must be tuned to select background cells by low RNA counts.
- The reference implementation is an R CRAN package; muon provides the Python wrapper within its protein module.
- MUON notes there is no open exchange format for multi-omics data across languages other than its HDF5 serialisation, so .h5mu is the interoperable output.

## Evidence

- [methods] pt.pp.dsb with the raw protein matrix as the empty/background reference: "pt.pp.dsb(adata_prot, adata_prot_raw, empty_counts_range=...)"
- [readme] dsb purpose stated in its README: [evidence span withheld — anchor status: external_doc]
- [readme] dsb deconvolves ADT noise sources (Mulè, Martins, and Tsang 2022): "describes
our deconvolution of ADT noise sources and development of dsb."
- [discussion] HDF5 serialisation gives cross-language access to MuData objects: "Serialisation to HDF5 makes MuData objects accessible to other programming languages, including R and Julia."
- [methods] muon reads Cell Ranger HDF5 matrices: "with {func}`muon.read_10x_h5`"
