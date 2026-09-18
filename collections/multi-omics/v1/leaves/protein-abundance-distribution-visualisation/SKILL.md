---
name: protein-abundance-distribution-visualisation
description: Use when CITE-seq 'Antibody capture' counts have been dsb-normalised
  and you need to inspect per-epitope abundance distributions before defining cell
  types or integrating protein with gene expression.
when_to_use_negative:
- The data are gene-expression counts or ATAC fragments rather than CITE-seq 'Antibody
  capture' (ADT) counts — other modalities use different preprocessing (e.g. ac.pp.tfidf
  for ATAC).
- Protein counts are already normalised (e.g. via pt.pp.clr) or already background-corrected
  — re-running dsb and re-plotting would double-correct the signal.
- No raw/unfiltered matrix is available to define the empty-droplet background required
  by dsb.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_4010
tools:
- name: MUON
  role: Reads 10x HDF5 matrices (muon.read_10x_h5), hosts the protein modality in
    MuData, and provides pt.pp.dsb for denoising-and-scaling of ADT counts before
    plotting
  repo: https://github.com/scverse/muon
- name: SCANPY
  role: Single-cell plotting/exploratory utilities used to visualise per-feature distributions
    of normalised protein abundances
  repo: https://github.com/scverse/muon
- name: dsb
  role: Original reference implementation of the denoising-and-scaling normalisation
    that muon's pt.pp.dsb wraps
  repo: https://github.com/niaid/dsb
- name: NumPy
  role: Array operations for extracting and validating the normalised protein matrix
    (e.g. finiteness checks)
- name: pandas
  role: Data handling for per-feature abundance values during plotting and validation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:5f58dc2e7a010fa4c2504cdb433912912ce7960492a0404e4e7148c13518b125
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_4010
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# protein-abundance-distribution-visualisation

## Summary

Plot per-feature distributions of dsb-normalised CITE-seq protein (epitope) abundances, mirroring Fig. 3c of the MUON paper, so that background-corrected protein signals can be inspected for cell-type definition akin to flow-cytometry gating.

## When to use

Use when CITE-seq 'Antibody capture' counts have been dsb-normalised and you need to inspect per-epitope abundance distributions before defining cell types or integrating protein with gene expression.

## When NOT to use

- The data are gene-expression counts or ATAC fragments rather than CITE-seq 'Antibody capture' (ADT) counts — other modalities use different preprocessing (e.g. ac.pp.tfidf for ATAC).
- Protein counts are already normalised (e.g. via pt.pp.clr) or already background-corrected — re-running dsb and re-plotting would double-correct the signal.
- No raw/unfiltered matrix is available to define the empty-droplet background required by dsb.

## Inputs

- Filtered Cell Ranger 10x HDF5 feature-barcode matrix (.h5) containing cell-associated counts
- Raw (unfiltered) Cell Ranger 10x HDF5 matrix used to identify empty/background droplets
- Protein ('Antibody capture') AnnData modality extracted from MuData

## Outputs

- Normalised protein modality written to an .h5mu file
- Per-feature distribution plots of dsb-normalised protein abundances for all measured epitopes (Fig. 3c-style)

## How to apply

First read the filtered and raw Cell Ranger HDF5 matrices with muon.read_10x_h5, keeping both cell-associated counts and the raw matrix whose low-RNA-count droplets serve as the empty/background reference. Extract the 'Antibody capture' AnnData modality from the MuData objects and run pt.pp.dsb with the raw protein matrix as background, setting empty_counts_range to select empty droplets by low RNA counts — this estimates and removes ambient and non-specific background signal from the ADT counts. Save the normalised protein modality to an .h5mu file, then plot per-feature distributions of the normalised protein abundances for all measured epitopes (e.g. with scanpy/muon plotting utilities), mirroring Fig. 3c. Validation: confirm the .h5mu exists and the normalised protein matrix contains finite, background-corrected values for every antibody feature.

## Related tools

- **MUON** (Reads 10x HDF5 matrices (muon.read_10x_h5), hosts the protein modality in MuData, and provides pt.pp.dsb for denoising-and-scaling of ADT counts before plotting) — https://github.com/scverse/muon
- **SCANPY** (Single-cell plotting/exploratory utilities used to visualise per-feature distributions of normalised protein abundances) — https://github.com/scverse/muon
- **dsb** (Original reference implementation of the denoising-and-scaling normalisation that muon's pt.pp.dsb wraps) — https://github.com/niaid/dsb
- **NumPy** (Array operations for extracting and validating the normalised protein matrix (e.g. finiteness checks))
- **pandas** (Data handling for per-feature abundance values during plotting and validation)

## Examples

```
import muon as mu; mdata = mu.read_10x_h5('filtered_feature_bc_matrix.h5'); mdata_raw = mu.read_10x_h5('raw_feature_bc_matrix.h5'); prot = mdata['prot']; mu.prot.pp.dsb(prot, mdata_raw['prot'], empty_counts_range=(1, 20))
```

## Evaluation signals

- The output .h5mu file exists and is readable as a MuData with a protein modality.
- The normalised protein matrix contains finite (no NaN/Inf) background-corrected values for every antibody feature.
- Empty-droplet background was estimated from the raw matrix using empty_counts_range selecting low RNA-count droplets.
- Per-feature distribution plots cover all measured epitopes and resemble the Fig. 3c feature-distribution layout.
- Normalised values are usable for cell-type definition akin to flow-cytometry gating (interpretable spread across cell populations).

## Limitations

- dsb requires a raw/unfiltered matrix so background droplets can be selected via empty_counts_range; without it, the method cannot estimate background.
- empty_counts_range must be set appropriately for the dataset (low RNA counts defining empty droplets) — a poor range yields incorrect background estimates.
- dsb normalisation is specific to antibody-derived-tag data from CITE-seq-like assays (CITE-seq, ASAP-seq, TEA-seq); it is not a general-purpose normalisation.

## Evidence

- [methods] dsb normalisation with raw background: "pt.pp.dsb(adata_prot, adata_prot_raw, empty_counts_range=...)"
- [methods] reading 10x data with muon: "with {func}`muon.read_10x_h5`"
- [readme] dsb purpose from README: [evidence span withheld — anchor status: external_doc]
- [readme] CITE-seq tutorial reference: [evidence span withheld — anchor status: external_doc]
- [discussion] HDF5 serialisation interoperability: "Serialisation to HDF5 makes MuData objects accessible to other programming languages, including R and Julia."
