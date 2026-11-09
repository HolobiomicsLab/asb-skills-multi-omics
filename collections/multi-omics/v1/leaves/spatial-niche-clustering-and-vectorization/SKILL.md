---
name: spatial-niche-clustering-and-vectorization
description: Use when a dataset contains an aligned H&E (or other staining) image
  layer alongside single-cell spatial transcriptomics and the goal is to identify
  and interpret image-derived spatial niches and their cell-type enrichments.
when_to_use_negative:
- Input data lacks an image/staining layer — the Sopa discussion notes that without
  the protein-staining layer, extracting the B cell population in the Xenium data
  would not have been possible
- You only need transcript-based neighborhood clustering rather than image-morphology-derived
  niches (consider tools like Novae instead)
- The H&E image is not aligned to transcriptomic coordinates and landmark alignment
  cannot be performed — embeddings would be computed from wrong cell locations
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Sopa
  role: Provides compute_embeddings (ResNet on 250x250 patches), cluster_embeddings,
    vectorize_niches, niches_geometry_stats, and aggregate APIs
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Unified data framework holding images, transcripts, and cell tables used by
    Sopa for niche analysis
  repo: https://github.com/scverse/spatialdata
- name: Xenium
  role: Source platform of the pancreatic cancer dataset (transcripts, protein staining,
    H&E) on which niches are computed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:2ab60676b18412db0e7e4fb84ff506f437ad2075d5733937656beae0f2777fa9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - MERSCOPE
  - Sopa
  - SpatialData
  - Xenium
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# spatial-niche-clustering-and-vectorization

## Summary

Derive H&E/image-derived spatial niches by computing per-cell embeddings from image patches (e.g., 250x250 H&E patches via a pre-trained ResNet in Sopa), clustering them into niches, then vectorizing niche geometries and characterizing them with geometric/spatial statistics to gain interpretability from H&E layers in spatial omics data.

## When to use

Use when a dataset contains an aligned H&E (or other staining) image layer alongside single-cell spatial transcriptomics and the goal is to identify and interpret image-derived spatial niches and their cell-type enrichments.

## When NOT to use

- Input data lacks an image/staining layer — the Sopa discussion notes that without the protein-staining layer, extracting the B cell population in the Xenium data would not have been possible
- You only need transcript-based neighborhood clustering rather than image-morphology-derived niches (consider tools like Novae instead)
- The H&E image is not aligned to transcriptomic coordinates and landmark alignment cannot be performed — embeddings would be computed from wrong cell locations

## Inputs

- SpatialData object containing per-cell aggregated transcript counts and staining-channel intensities (including TROP2)
- H&E image layer aligned to transcriptomic coordinates (transformation matrix .csv from Xenium Explorer landmark alignment)
- Cell segmentation boundaries / cell-type annotations for enrichment analysis

## Outputs

- Per-cell H&E-derived niche cluster assignments
- Vectorized niche polygons (sopa.spatial.vectorize_niches)
- Niche geometry statistics (sopa.spatial.niches_geometry_stats)
- Per-niche cell-type enrichment and TROP2 intensity table (e.g., niche_enrichment_trop2.csv)

## How to apply

After aggregating transcripts and staining intensities per cell with sopa.aggregate, compute per-cell image embeddings from 250x250 H&E patches centered on each cell using sopa.compute_embeddings with a pre-trained ResNet. Cluster the embeddings into spatial niches with sopa.cluster_embeddings, then convert niche regions to polygons with sopa.spatial.vectorize_niches and quantify their geometry with sopa.spatial.niches_geometry_stats. Interpret niches by computing per-niche cell-type enrichments and marker/protein specificity (e.g., TROP2 intensity per niche) over all N=175,022 cells, writing results such as niche_enrichment_trop2.csv. Alignment of the H&E layer to transcriptomic coordinates (via Xenium Explorer landmark-based alignment exported as a .csv transformation matrix and applied with sopa.io.explorer.align) is a prerequisite so patches map to the correct cells.

## Related tools

- **Sopa** (Provides compute_embeddings (ResNet on 250x250 patches), cluster_embeddings, vectorize_niches, niches_geometry_stats, and aggregate APIs) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Unified data framework holding images, transcripts, and cell tables used by Sopa for niche analysis) — https://github.com/scverse/spatialdata
- **Xenium** (Source platform of the pancreatic cancer dataset (transcripts, protein staining, H&E) on which niches are computed)

## Examples

```
sdata = sopa.compute_embeddings(sdata, 'he_image', patch_width=250); sopa.cluster_embeddings(sdata, [0]); sopa.spatial.vectorize_niches(sdata)
```

## Evaluation signals

- Per-niche cell-type enrichment matches expected biological patterns: niche 3 specific to Acinar cells, niche 5 to Ductal cells with TM4SF4/APCDD1 specificity, niche 4 enriched in B and Myeloid cells
- TROP2 expression/intensity is higher in tumour-specific niches (0 and 3) than other niches
- Niches cover all segmented cells (N=175,022 in the Xenium pancreatic cancer dataset) with valid polygon geometries after vectorization
- H&E patches correspond to correctly aligned cell coordinates after applying the Xenium Explorer transformation matrix via sopa.io.explorer.align

## Limitations

- Sopa relies on prior segmentation; cells missed by segmentation cannot be recovered by downstream niche clustering
- Current spatial technologies provide either proteins or transcripts; extra information layers remain a future development
- H&E-based prediction of refined spatial-transcriptomics cell types is only proposed as future work, not implemented
- Without a protein-staining layer, some cell populations (e.g., B cells in the Xenium data) cannot be extracted, limiting niche interpretability

## Evidence

- [results] niche 3 is highly specific to Acinar cells, niche 5 to Ductal cells (with TM4SF4 and APCDD1 specificity), and niche 4 is enriched in B cells and Myeloid cells: [evidence span withheld — anchor status: not_found]
- [methods] Compute per-cell embeddings from 250x250 H&E patches around each cell using sopa.compute_embeddings with a pre-trained ResNet: [evidence span withheld — anchor status: not_found]
- [abstract] Built upon the universal SpatialData framework, Sopa optimizes tasks like segmentation, transcript/channel aggregation, annotation, and geometric/spatial analysis.: "Built upon the universal SpatialData framework, Sopa optimizes tasks like segmentation, transcript/channel aggregation, annotation, and geometric/spatial analysis."
- [discussion] Without this protein layer, extracting the B cell population in the Xenium data would not have been possible.: "Without this protein layer, extracting the B cell population in the Xenium data would not have been possible."
