---
name: gene-expression-and-accessibility-factor-visualization
description: Use when a MuData object with paired scRNA-seq and scATAC-seq modalities
  already contains a fitted latent factor space (e.g. X_mofa in obsm) and you need
  to interpret what each primary factor represents biologically.
when_to_use_negative:
- The MuData object has no fitted multimodal factors yet — run mu.tl.mofa first; plotting
  factors before confirming "X_mofa" in mdata.obsm would visualise an empty or missing
  latent space
- The analysis goal is single-omics (RNA-only) dimensionality reduction, where single-modality
  scanpy visualisation suffices and ATAC promoter accessibility is irrelevant
- The input lacks the 'highly_variable' feature column, since muon's MOFA wrapper
  by default trains on use_var='highly_variable' and factors would not reflect the
  filtered feature space being visualised
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: MUON
  role: Provides the MuData structure, mu.tl.mofa factor fitting, and the plotting/visualisation
    framework for multimodal omics data
  repo: https://github.com/scverse/muon
- name: MOFA+
  role: Statistical framework for comprehensive integration of multi-modal single-cell
    data whose leading factors are visualised against marker genes and promoter accessibility
- name: SCANPY
  role: Performs RNA normalisation (sc.pp.normalise_per_cell with counts_per_cell_after=1e4,
    sc.pp.log1p) and supports marker expression plotting along factors
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:ae86be3537d341ff10f943ddd04df31641c5bfd37acb9cdab795a5d46796b958
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/muon
  tools:
  - MOFA+
  - MUON
  - SCANPY
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# gene-expression-and-accessibility-factor-visualization

## Summary

Visualise the biological axes captured by multimodal factor analysis by plotting marker gene expression (RNA modality) and promoter accessibility (ATAC modality) along the leading learned factors of a MuData object, confirming that factors capture axes such as the myeloid—lymphoid axis and cytotoxicity.

## When to use

Use when a MuData object with paired scRNA-seq and scATAC-seq modalities already contains a fitted latent factor space (e.g. X_mofa in obsm) and you need to interpret what each leading factor represents biologically.

## When NOT to use

- The MuData object has no fitted multimodal factors yet — run mu.tl.mofa first; plotting factors before confirming "X_mofa" in mdata.obsm would visualise an empty or missing latent space
- The analysis goal is single-omics (RNA-only) dimensionality reduction, where single-modality scanpy visualisation suffices and ATAC promoter accessibility is irrelevant
- The input lacks the 'highly_variable' feature column, since muon's MOFA wrapper by default trains on use_var='highly_variable' and factors would not reflect the filtered feature space being visualised

## Inputs

- MuData object with 'rna' and 'atac' modalities read via mu.read_10x_h5 from filtered_feature_bc_matrix.h5
- mdata.obsm['X_mofa'] factor matrix produced by mu.tl.mofa, with factor loadings stored alongside
- Marker gene list (RNA modality) and promoter regions (ATAC modality) for the biological axes of interest

## Outputs

- Per-factor plots of marker gene expression (RNA) and promoter accessibility (ATAC) along leading factors
- Interpretation of leading factors as biological axes (myeloid—lymphoid axis, cytotoxicity)
- Saved MuData file mofa_pbmc_multiome.h5mu

## How to apply

After running mu.tl.mofa(mdata) on a PBMC multiome MuData object and confirming that "X_mofa" in mdata.obsm evaluates to True, extract the factor values from mdata.obsm['X_mofa'] with factor loadings stored alongside. For the RNA modality, plot normalised (sc.pp.normalise_per_cell with counts_per_cell_after=1e4 followed by sc.pp.log1p) expression of canonical marker genes along the leading factors. For the ATAC modality, plot promoter accessibility along the same factors to check that RNA and ATAC views agree. Interpretation succeeds when the leading factors align with canonical biological differences — specifically the myeloid—lymphoid axis and cytotoxicity — integrating variation from both modalities. Persist the interpreted object with mdata.write('mofa_pbmc_multiome.h5mu').

## Related tools

- **MUON** (Provides the MuData structure, mu.tl.mofa factor fitting, and the plotting/visualisation framework for multimodal omics data) — https://github.com/scverse/muon
- **MOFA+** (Statistical framework for comprehensive integration of multi-modal single-cell data whose leading factors are visualised against marker genes and promoter accessibility)
- **SCANPY** (Performs RNA normalisation (sc.pp.normalise_per_cell with counts_per_cell_after=1e4, sc.pp.log1p) and supports marker expression plotting along factors)

## Examples

```
import muon as mu; mdata = mu.read_10x_h5('filtered_feature_bc_matrix.h5'); mu.tl.mofa(mdata); assert "X_mofa" in mdata.obsm; mdata.write('mofa_pbmc_multiome.h5mu')
```

## Evaluation signals

- Assert that "X_mofa" in mdata.obsm evaluates to True before any factor plotting
- Leading factors should show concordant trends between marker gene expression (RNA) and promoter accessibility (ATAC) for the same cell populations
- Visualised factors should reproduce canonical biological differences — the myeloid—lymphoid axis and cytotoxicity — rather than technical variation
- The interpreted object writes successfully with mdata.write('mofa_pbmc_multiome.h5mu') and reloads with both modalities and factors intact

## Limitations

- Factor interpretation depends on prior feature selection: muon's MOFA wrapper by default trains on use_var='highly_variable', so factors may miss biology in excluded features
- MUON's emerging-assay support and explicit modelling of dependencies between feature sets across omics are future work, so factor plots cannot capture cross-omics dependency structure directly

## Evidence

- [results] Plot marker gene expression (RNA modality) and promoter accessibility (ATAC modality) along the leading learned factors to inspect the captured axes of variation.: [evidence span withheld — anchor status: not_found]
- [results] factors capture canonical biological differences: [evidence span withheld — anchor status: not_found]
- [methods] X_mofa check: [evidence span withheld — anchor status: not_found]
- [abstract] muon visualisation scope: "we present a data standard and an analysis framework for multi-omics, MUON, designed to organise, analyse, visualise, and exchange multimodal data"
- [readme] muon tutorials readme: [evidence span withheld — anchor status: external_doc]
