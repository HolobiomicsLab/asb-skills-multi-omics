---
name: he-image-patch-embedding-computation
description: Use when a SpatialData object contains both segmented cells with transcript
  counts and an aligned H&E image, and the goal is to derive morphological niches
  from H&E rather than expression alone.
when_to_use_negative:
- No H&E (or other morphology) image layer is present or it cannot be aligned to transcript
  coordinates — embeddings cannot be computed from expression alone
- Cell segmentation masks are missing or of low quality — patches centered on unreliable
  boundaries produce meaningless morphology embeddings
- The goal is transcript-only spatial domain detection (e.g., Novae-style neighborhood
  clustering) where morphology integration is unnecessary
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Sopa
  role: Provides sopa.compute_embeddings, sopa.cluster_embeddings, sopa.spatial.vectorize_niches,
    sopa.spatial.niches_geometry_stats, sopa.aggregate and sopa.io.explorer.align
    used to build, cluster, and validate H&E patch embeddings
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Unified data framework holding the transcripts, protein-staining and H&E layers
    in a single object that embeddings are computed over
  repo: https://github.com/scverse/spatialdata
- name: Xenium
  role: Platform supplying the pancreatic cancer transcriptomics and protein-staining
    data over which H&E patches are aligned and embedded
- name: Xenium Explorer
  role: Landmark-based alignment of the H&E layer to transcript coordinates; exports
    the transformation matrix as .csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:d231263e337dba11e007be3903469bd26b6ae9ceea9d787958e3d661cc033934
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - MERSCOPE
  - Sopa
  - SpatialData
  - Xenium
  - Xenium Explorer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# he-image-patch-embedding-computation

## Summary

Compute per-cell morphology embeddings by extracting H&E image patches centered on segmented cell boundaries and embedding them with a pre-trained CNN (e.g., ResNet), enabling H&E-derived spatial niche discovery within the Sopa/SpatialData framework on image-based spatial-omics data such as Xenium pancreatic cancer.

## When to use

Use when a SpatialData object contains both segmented cells with transcript counts and an aligned H&E image, and the goal is to derive morphological niches from H&E rather than expression alone.

## When NOT to use

- No H&E (or other morphology) image layer is present or it cannot be aligned to transcript coordinates — embeddings cannot be computed from expression alone
- Cell segmentation masks are missing or of low quality — patches centered on unreliable boundaries produce meaningless morphology embeddings
- The goal is transcript-only spatial domain detection (e.g., Novae-style neighborhood clustering) where morphology integration is unnecessary

## Inputs

- SpatialData object with cell segmentation boundaries and transcript counts
- Aligned H&E histology image layer (transformation matrix exported as .csv from Xenium Explorer and applied via sopa.io.explorer.align)
- Cell coordinate table (N=175,022 cells in the Xenium pancreatic cancer example)

## Outputs

- Per-cell image embedding matrix (ResNet features per 250x250 H&E patch)
- Clustered spatial niche labels per cell
- Vectorized niche polygons and niche geometry statistics (sopa.spatial.vectorize_niches, sopa.spatial.niches_geometry_stats)
- Per-niche cell-type enrichment and TROP2 intensity table (e.g., niche_enrichment_trop2.csv)

## How to apply

First align the H&E layer to the transcriptomic coordinates using landmark-based alignment in Xenium Explorer (or napari-spatialdata), exporting the transformation matrix as a .csv and applying it with sopa.io.explorer.align so H&E patches fall on the correct cells. For each segmented cell, extract a 250x250 H&E patch centered on the cell and embed it with sopa.compute_embeddings using a pre-trained ResNet; the resulting per-cell embeddings encode morphology rather than transcriptomics. Cluster the embeddings with sopa.cluster_embeddings to define spatial niches, then use sopa.spatial.vectorize_niches and sopa.spatial.niches_geometry_stats to vectorize and characterize them. Finally aggregate per-cell transcript counts and staining-channel intensities (e.g., TROP2) with sopa.aggregate over niche-assigned cells so niches can be validated against cell-type and protein enrichments. Correctness is judged by whether niches show coherent, specific enrichments (e.g., acinar-, ductal-, or immune-enriched niches in the Xenium pancreatic cancer data) rather than arbitrary spatial partitioning.

## Related tools

- **Sopa** (Provides sopa.compute_embeddings, sopa.cluster_embeddings, sopa.spatial.vectorize_niches, sopa.spatial.niches_geometry_stats, sopa.aggregate and sopa.io.explorer.align used to build, cluster, and validate H&E patch embeddings) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Unified data framework holding the transcripts, protein-staining and H&E layers in a single object that embeddings are computed over) — https://github.com/scverse/spatialdata
- **Xenium** (Platform supplying the pancreatic cancer transcriptomics and protein-staining data over which H&E patches are aligned and embedded)
- **Xenium Explorer** (Landmark-based alignment of the H&E layer to transcript coordinates; exports the transformation matrix as .csv)

## Examples

```
import sopa; sdata = sopa.io.explorer.align(sdata, 'alignment_matrix.csv'); embeddings = sopa.compute_embeddings(sdata, patch_width=250, model='resnet'); sopa.cluster_embeddings(sdata, embeddings)
```

## Evaluation signals

- Each cell has exactly one embedding vector corresponding to its 250x250 H&E patch (1:1 cell-to-embedding correspondence over all N=175,022 cells)
- Clustered niches are spatially contiguous and their vectorized boundaries pass niches_geometry_stats sanity checks (reasonable area/geometry per niche)
- Niches show specific, interpretable cell-type enrichments (e.g., niche 3 acinar-enriched, niche 5 ductal with TM4SF4/APCDD1, niche 4 B-cell/myeloid-enriched)
- Staining intensity aggregated per niche is coherent — e.g., TROP2 higher in tumour-specific niches (niches 0 and 3) — confirming alignment of H&E to transcript coordinates was correct
- B cells are recoverable in the analysis only when the protein-staining layer is aggregated alongside; absence of B-cell annotation indicates the aggregation step was skipped or misconfigured

## Limitations

- Embedding quality depends entirely on correct H&E-to-transcript alignment; misalignment shifts patches off cells and corrupts niche assignments
- Interpretability of H&E-derived niches relies on downstream aggregation of transcripts and protein stains — embeddings alone yield unlabeled clusters
- B cell extraction in the Xenium data depended on the protein-staining layer; without it this population cannot be recovered
- H&E-based prediction of refined spatial-transcriptomics cell types from embeddings is only proposed as future work and is not implemented in Sopa
- Method is tied to image-based spatial-omics platforms (Xenium, MERSCOPE, CosMX); it is not applicable to non-image-based spatial data

## Evidence

- [discussion] ResNet embedding for niche clustering: [evidence span withheld — anchor status: not_found]
- [abstract] technology-invariant pipeline on SpatialData: "Built upon the universal SpatialData framework, Sopa optimizes tasks like segmentation, transcript/channel aggregation, annotation, and geometric/spatial analysis."
- [discussion] protein layer required for B cells: "Without this protein layer, extracting the B cell population in the Xenium data would not have been possible."
- [discussion] future H&E cell-type prediction: "or also build upon Sopa to develop tools that predict refined spatial-transcriptomics cell types based on H&E images"
