---
name: spatial-omics-multilayer-image-alignment
description: Use when a spatial omics run provides an H&E (or protein-channel) image
  plus transcript coordinates in different frames, and you need per-cell morphology-derived
  features (e.g., ResNet embeddings of 250x250 H&E patches) co-registered with transcript
  counts.
when_to_use_negative:
- The transcriptomics data already contains a usable protein-staining or H&E layer
  in the native coordinate system, so no cross-modality alignment is needed
- You only need transcript-based segmentation/annotation (e.g., Baysor/Tangram) with
  no morphology layer — Sopa's aggregation and alignment machinery adds no value
- The tissue section lacks landmarks visible in both H&E and the spatial omics channels,
  making landmark-based alignment unreliable
edam_operation: http://edamontology.org/operation_0337
tools:
- name: Sopa
  role: Pipeline that reads all layers into SpatialData, applies sopa.io.explorer.align
    with the transformation matrix, computes ResNet embeddings of H&E patches, and
    aggregates counts/intensities per cell
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Unified data framework holding the transcripts, H&E image, and staining channels
    so alignment and aggregation operate on one object
  repo: https://github.com/scverse/spatialdata
- name: Xenium
  role: Source technology providing the pancreatic cancer transcriptomics plus protein-staining
    data that the H&E layer is aligned to
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:2b7985046d1a6b64fb210836c1b9a4973a5c59bb94f8888ff7b31c8161f1d608
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - MERSCOPE
  - Sopa
  - SpatialData
  - Xenium
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# spatial-omics-multilayer-image-alignment

## Summary

Aligns a brightfield H&E image layer to the transcriptomic coordinate system of an image-based spatial omics run (e.g., Xenium pancreatic cancer) inside a single SpatialData object via Sopa, so that per-cell H&E patches, protein staining intensities, and transcript counts can be jointly analyzed.

## When to use

Use when a spatial omics run provides an H&E (or protein-channel) image plus transcript coordinates in different frames, and you need per-cell morphology-derived features (e.g., ResNet embeddings of 250x250 H&E patches) co-registered with transcript counts.

## When NOT to use

- The transcriptomics data already contains a usable protein-staining or H&E layer in the native coordinate system, so no cross-modality alignment is needed
- You only need transcript-based segmentation/annotation (e.g., Baysor/Tangram) with no morphology layer — Sopa's aggregation and alignment machinery adds no value
- The tissue section lacks landmarks visible in both H&E and the spatial omics channels, making landmark-based alignment unreliable

## Inputs

- Xenium (or MERSCOPE/CosMX) transcript coordinates and per-cell segmentation outputs
- Brightfield H&E image of the same tissue section
- Protein-channel staining images (e.g., TROP2, B-cell markers)
- Landmark transformation matrix exported from Xenium Explorer or napari-spatialdata as a .csv file

## Outputs

- A single SpatialData object containing aligned H&E, staining, and transcript layers
- Per-cell H&E patch embeddings (pre-trained ResNet, 250x250 patches)
- Per-cell aggregated transcript counts and staining intensities (niche_enrichment_trop2.csv with per-niche enrichments and TROP2 distribution)

## How to apply

Read the raw layers — transcripts, protein-channel staining, and the H&E image — with the appropriate sopa.io readers into one SpatialData object, since Sopa converts any data into a SpatialData object on which all downstream tasks are performed. Perform landmark-based alignment of the H&E layer to the transcriptomic coordinates using the Xenium Explorer (or napari-spatialdata), export the resulting transformation matrix as a .csv file, and apply it with sopa.io.explorer.align. Once aligned, compute per-cell embeddings from 250x250 H&E patches around each cell (sopa.compute_embeddings with a pre-trained ResNet) and aggregate per-cell transcript counts and staining-channel intensities (including markers such as TROP2) with sopa.aggregate, which averages staining intensities and counts transcripts inside each cell. Judge success by whether niche assignment of H&E-embedding clusters yields biologically coherent cell-type enrichments (e.g., acinar-enriched niche 3, ductal niche 5, B/myeloid-enriched niche 4) over the N=175,022 cells.

## Related tools

- **Sopa** (Pipeline that reads all layers into SpatialData, applies sopa.io.explorer.align with the transformation matrix, computes ResNet embeddings of H&E patches, and aggregates counts/intensities per cell) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Unified data framework holding the transcripts, H&E image, and staining channels so alignment and aggregation operate on one object) — https://github.com/scverse/spatialdata
- **Xenium** (Source technology providing the pancreatic cancer transcriptomics plus protein-staining data that the H&E layer is aligned to)

## Examples

```
import sopa; sdata = sopa.io.explorer.align(sdata, align_file='landmarks_transformation.csv')
```

## Evaluation signals

- The SpatialData object contains all three layers (transcripts, H&E, protein channels) and sopa.io.explorer.align completes without coordinate errors
- Per-cell H&E patch embeddings cluster into niches whose cell-type enrichments match the published pattern (niche 3 acinar, niche 5 ductal with TM4SF4/APCDD1, niche 4 B/myeloid)
- TROP2 intensity distribution shows higher expression in tumour-specific niches (0 and 3) after aggregation over aligned cells
- Cell counts in the aligned analysis match the expected N=175,022 cells, with no cells dropped by misalignment
- Output CSV (niche_enrichment_trop2.csv) contains per-niche cell-type enrichment and TROP2 intensity columns with plausible ranges

## Limitations

- B cell extraction depended on availability of a protein-staining layer; without it, that population could not be extracted in the Xenium data
- Predicting refined spatial-transcriptomics cell types directly from H&E images is only proposed as future work, not implemented in Sopa
- Alignment quality depends on landmarks selected interactively in Xenium Explorer or napari-spatialdata; poor landmarks yield misaligned H&E patches and noisy embeddings

## Evidence

- [abstract] Built upon the universal SpatialData framework, Sopa optimizes tasks like segmentation, transcript/channel aggregation, annotation, and geometric/spatial analysis.: "Built upon the universal SpatialData framework, Sopa optimizes tasks like segmentation, transcript/channel aggregation, annotation, and geometric/spatial analysis."
- [intro] Sopa converts any data into a SpatialData object, on which all of the six following tasks are performed.: "Sopa converts any data into a SpatialData object, on which all of the six following tasks are performed."
- [intro] we average the staining intensities and count the transcripts inside each cell, allowing further tasks such as annotation.: "we average the staining intensities and count the transcripts inside each cell, allowing further tasks such as annotation."
- [discussion] Without this protein layer, extracting the B cell population in the Xenium data would not have been possible.: "Without this protein layer, extracting the B cell population in the Xenium data would not have been possible."
- [methods] Align the H&E layer to the transcriptomic coordinates using landmark-based alignment via the Xenium Explorer (or napari-spatialdata), exporting the transformation matrix as a .csv file and applying it with sopa.io.explorer.align.: [evidence span withheld — anchor status: not_found]
