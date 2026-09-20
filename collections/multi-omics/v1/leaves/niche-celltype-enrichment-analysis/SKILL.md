---
name: niche-celltype-enrichment-analysis
description: Use when spatial niches have been clustered from image embeddings (e.g.,
  H&E ResNet patches) and you need to interpret them via cell-type enrichment and
  protein/marker specificity (e.g., TROP2, TM4SF4, APCDD1).
when_to_use_negative:
- The data lacks a protein-staining layer and cell types cannot be resolved — the
  article notes that without the protein layer, extracting the B cell population in
  the Xenium data would not have been possible
- Niches are derived from transcriptomics-only community detection (e.g., Novae) rather
  than image embeddings, so image-to-transcript alignment is unnecessary
- Input is already a per-niche enrichment table and the task is downstream visualization
  rather than computation
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Sopa
  role: Computes H&E patch embeddings, clusters them into niches, aggregates transcripts/staining
    per cell, and provides vectorize_niches and niches_geometry_stats for niche characterization
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Unified data representation holding the aligned H&E layer, transcripts, and
    aggregated cell table for niche analysis
  repo: https://github.com/scverse/spatialdata
- name: Xenium
  role: Source platform providing the pancreatic cancer transcriptomics and protein-staining
    data with H&E layer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:f3378c4045f2a3d81e6c71b15e75ff606cc61e7cb4f67714c4df57433205577d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - MERSCOPE
  - Sopa
  - SpatialData
  - Xenium
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# niche-celltype-enrichment-analysis

## Summary

Characterize image-derived spatial niches by computing per-niche cell-type enrichments and marker/protein intensity distributions, coupling H&E-derived niche labels with transcript and staining aggregates from Sopa on the Xenium pancreatic cancer dataset (N=175,022 cells).

## When to use

Use when spatial niches have been clustered from image embeddings (e.g., H&E ResNet patches) and you need to interpret them via cell-type enrichment and protein/marker specificity (e.g., TROP2, TM4SF4, APCDD1).

## When NOT to use

- The data lacks a protein-staining layer and cell types cannot be resolved — the article notes that without the protein layer, extracting the B cell population in the Xenium data would not have been possible
- Niches are derived from transcriptomics-only community detection (e.g., Novae) rather than image embeddings, so image-to-transcript alignment is unnecessary
- Input is already a per-niche enrichment table and the task is downstream visualization rather than computation

## Inputs

- SpatialData object with aligned H&E image layer, transcripts, and protein-channel staining (Xenium pancreatic cancer data)
- Per-cell H&E ResNet embeddings clustered into niche labels (from sopa.cluster_embeddings)
- Per-cell transcript count matrix and staining intensities (from sopa.aggregate)
- Cell-type annotations per cell (e.g., Acinar, Ductal, B cells, Myeloid)

## Outputs

- Per-niche cell-type enrichment table (niche x cell-type frequencies/enrichments)
- Per-niche marker/protein intensity distributions (e.g., TROP2 per niche; TM4SF4, APCDD1 specificity)
- niche_enrichment_trop2.csv results file
- Vectorized niche geometries and geometry statistics (sopa.spatial.vectorize_niches, niches_geometry_stats)

## How to apply

After computing per-cell H&E embeddings from 250x250 patches with sopa.compute_embeddings (pre-trained ResNet) and clustering them with sopa.cluster_embeddings, vectorize and quantify the niches using sopa.spatial.vectorize_niches and sopa.spatial.niches_geometry_stats. Aggregate per-cell transcript counts and staining-channel intensities (including TROP2) onto the niche-assigned cells with sopa.aggregate, operating on a single SpatialData object whose H&E layer was aligned to transcriptomic coordinates via a landmark-based transformation matrix exported from Xenium Explorer and applied with sopa.io.explorer.align. Then compute cell-type composition per niche and compare observed versus expected cell-type frequencies to identify specific enrichments (e.g., niche 3 acinar-enriched, niche 5 ductal with TM4SF4/APCDD1 specificity, niche 4 B-cell/myeloid enriched), plus per-niche TROP2 intensity distributions. Write the enrichment and intensity results to a table such as niche_enrichment_trop2.csv so niche identities become biologically interpretable.

## Related tools

- **Sopa** (Computes H&E patch embeddings, clusters them into niches, aggregates transcripts/staining per cell, and provides vectorize_niches and niches_geometry_stats for niche characterization) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Unified data representation holding the aligned H&E layer, transcripts, and aggregated cell table for niche analysis) — https://github.com/scverse/spatialdata
- **Xenium** (Source platform providing the pancreatic cancer transcriptomics and protein-staining data with H&E layer)

## Examples

```
import sopa; sdata = sopa.io.explorer.align(sdata, 'transformation_matrix.csv'); sopa.aggregate(sdata); sopa.spatial.vectorize_niches(sdata); sopa.spatial.niches_geometry_stats(sdata)
```

## Evaluation signals

- Niche assignments cover the expected number of cells (N=175,022) with each cell assigned to exactly one niche
- Known enrichments are recovered: niche 3 specific to Acinar cells, niche 5 to Ductal cells (TM4SF4 and APCDD1 specificity), niche 4 enriched in B cells and Myeloid cells
- TROP2 intensity is higher in tumour-specific niches (e.g., niches 0 and 3) than in stromal/immune niches
- niche_enrichment_trop2.csv is produced with per-niche rows containing cell-type enrichment values and TROP2 intensity summaries
- H&E-derived niches align spatially with transcriptomic coordinates after applying the Xenium Explorer transformation matrix

## Limitations

- B-cell extraction depended on the availability of a protein-staining layer; without it this population could not be resolved in the Xenium data
- H&E-based prediction of refined spatial-transcriptomics cell types is only proposed as future work, not implemented in Sopa
- Current spatial technologies provide either proteins or transcripts; extra information layers remain a future development

## Evidence

- [results] H&E-based niches show specific cell-type enrichments: niche 3 is highly specific to Acinar cells, niche 5 to Ductal cells (with TM4SF4 and APCDD1 specificity): [evidence span withheld — anchor status: not_found]
- [discussion] Without this protein layer, extracting the B cell population in the Xenium data would not have been possible.: "Without this protein layer, extracting the B cell population in the Xenium data would not have been possible."
- [abstract] Built upon the universal SpatialData framework, Sopa optimizes tasks like segmentation, transcript/channel aggregation, annotation, and geometric/spatial analysis.: "Built upon the universal SpatialData framework, Sopa optimizes tasks like segmentation, transcript/channel aggregation, annotation, and geometric/spatial analysis."
- [intro] we average the staining intensities and count the transcripts inside each cell, allowing further tasks such as annotation.: "we average the staining intensities and count the transcripts inside each cell, allowing further tasks such as annotation."
