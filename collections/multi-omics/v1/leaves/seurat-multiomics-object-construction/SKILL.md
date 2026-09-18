---
name: seurat-multiomics-object-construction
description: Use when a multimodal single-cell dataset is stored as .h5mu/MuData (e.g.
  PBMC RNA+ATAC) and must be loaded into R as MultiAssayExperiment or Seurat with
  one assay per modality.
when_to_use_negative:
- The data is already a native R object (AnnData-equivalent or SingleCellExperiment);
  no cross-language HDF5 exchange is needed.
- Only a single omics modality is present with no .mod group — a plain Seurat/SCE
  reader suffices instead of multimodal reconstruction.
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MUON
  role: Defines the MuData/HDF5 file standard being read from Python/R/Julia; its
    R bindings and in-place ops (intersect_obs) keep shared observations across modalities
  repo: https://github.com/scverse/muon
- name: MuData
  role: Container format (.h5mu) whose obs/obsm/var/varm and .mod layout is traversed
    to build R objects
  repo: https://github.com/scverse/muon
- name: MultiAssayExperiment
  role: 'Target R container: ExperimentList with one experiment per modality (''rna'',
    ''atac'') and mapped shared cell barcodes'
- name: Seurat
  role: 'Alternative R container: one Seurat assay per modality built from the same
    .h5mu stores'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13059-021-02577-8
    title: 'MUON: multimodal omics analysis framework'
  source_content_hash: sha256:9fa84074e67b9dee2fbdca085d65bef6a945e8062fd3529152415d7f227bb5f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/muon
  tools:
  - MUON
  - MuData
  - MultiAssayExperiment
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3
n_claims: 10
---

# seurat-multiomics-object-construction

## Summary

Reconstructs a MuData .h5mu container (HDF5 layout with obs/obsm/var/varm groups plus a .mod group of per-modality stores) into native R MultiAssayExperiment or Seurat objects, one experiment/assay per modality, to enable cross-language exchange of multi-omics data.

## When to use

Use when a multimodal single-cell dataset is stored as .h5mu/MuData (e.g. PBMC RNA+ATAC) and must be loaded into R as MultiAssayExperiment or Seurat with one assay per modality.

## When NOT to use

- The data is already a native R object (AnnData-equivalent or SingleCellExperiment); no cross-language HDF5 exchange is needed.
- Only a single omics modality is present with no .mod group — a plain Seurat/SCE reader suffices instead of multimodal reconstruction.

## Inputs

- MuData .h5mu file (HDF5 layout: top-level obs/obsm/var/varm groups plus .mod group with per-modality stores)
- Shared cell barcodes across modalities (e.g. 'rna' and 'atac' stores)

## Outputs

- R MultiAssayExperiment with an ExperimentList containing one experiment per modality
- Seurat object with one assay per modality (optional equivalent representation)
- Serialised R object, e.g. mdata_pbmc.rds written via saveRDS

## How to apply

Open the .h5mu file and traverse its HDF5 layout: top-level obs/obsm/var/varm groups plus the .mod group that holds one store per modality (e.g. 'rna' and 'atac'). Build a MultiAssayExperiment whose ExperimentList contains one experiment per modality, mapping shared cell barcodes across modalities (or use muon.pp.intersect_obs upstream in Python to keep observations present in all modalities). Optionally build an equivalent Seurat object with one assay per modality from the same .h5mu stores. Verify that modality keys ('atac', 'rna') and the observation (cell) counts match the source container. Rationale: HDF5 serialisation of MuData is the interoperability mechanism MUON provides, since no open, multi-language exchange format for multi-omics data previously existed.

## Related tools

- **MUON** (Defines the MuData/HDF5 file standard being read from Python/R/Julia; its R bindings and in-place ops (intersect_obs) keep shared observations across modalities) — https://github.com/scverse/muon
- **MuData** (Container format (.h5mu) whose obs/obsm/var/varm and .mod layout is traversed to build R objects) — https://github.com/scverse/muon
- **MultiAssayExperiment** (Target R container: ExperimentList with one experiment per modality ('rna', 'atac') and mapped shared cell barcodes)
- **Seurat** (Alternative R container: one Seurat assay per modality built from the same .h5mu stores)

## Examples

```
mae <- readMuDataH5("pbmc.h5mu"); stopifnot(all(c("rna","atac") %in% names(mae))); saveRDS(mae, "mdata_pbmc.rds")
```

## Evaluation signals

- mdata_pbmc.rds exists and reloads in R without error via readRDS.
- The loaded R object contains both expected modality keys 'atac' and 'rna'.
- Observation (cell) counts per modality in the R object equal the counts in the source .h5mu container.
- Cell barcodes are consistent across modalities (shared barcodes mapped correctly between experiments in the ExperimentList / assays).

## Limitations

- Support for emerging multi-omics assays (e.g. genomic-coordinate based assays or assays with spatial coordinates) is future work for MUON, not a current capability.
- Explicitly modeling dependencies between feature sets across omics, or between multiple sets of multi-omics experiments, in MuData is planned for future development.

## Evidence

- [discussion] HDF5 serialisation for cross-language access: "Serialisation to HDF5 makes MuData objects accessible to other programming languages, including R and Julia."
- [intro] No pre-existing exchange format: "Additionally, there currently exists no open exchange format for sharing multi-omics datasets that is accessible from different programming languages."
- [abstract] MuData design: "MUON stores multimodal data in an efficient yet flexible and interoperable data structure"
- [readme] muon is a multimodal omics Python framework: "`muon` is a multimodal omics Python framework."
