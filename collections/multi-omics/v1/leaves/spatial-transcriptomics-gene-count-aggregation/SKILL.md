---
name: spatial-transcriptomics-gene-count-aggregation
description: Use when a cell segmentation (vendor or Baysor/Cellpose) already exists
  for MERSCOPE/Xenium transcripts and you need a cell-by-gene count matrix, e.g. prior
  to clustering, UMAP, or Calinski-Harabasz comparison of segmentations.
when_to_use_negative:
- The input is already an aggregated cell-by-gene count table — re-running aggregation
  would be redundant
- No cell segmentation exists yet — segmentation (e.g. Baysor on transcript patches)
  must precede aggregation
- The data are protein-staining-only (e.g. MACSima/PhenoCycler) without a transcript
  layer — gene counting does not apply
tools:
- name: Sopa
  role: Provides the aggregation command (sopa aggregate --aggregate-genes) that counts
    transcripts inside each cell and applies min-transcripts filtering
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Unified data framework holding the .zarr object with transcripts, shapes,
    and the aggregated AnnData table
  repo: https://github.com/scverse/spatialdata
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:3eed8d9e94713de9ba417426da9535252132e54bc550ca23f8b3085d204b9836
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - MERSCOPE
  - Sopa
  - SpatialData
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# spatial-transcriptomics-gene-count-aggregation

## Summary

Aggregate per-cell gene counts from imaging-based spatial transcriptomics data (MERSCOPE, Xenium) by counting transcripts inside each segmented cell using Sopa's aggregation step, with gene-name and minimum-transcript filtering to remove low-quality cells before downstream clustering.

## When to use

Use when a cell segmentation (vendor or Baysor/Cellpose) already exists for MERSCOPE/Xenium transcripts and you need a cell-by-gene count matrix, e.g. prior to clustering, UMAP, or Calinski-Harabasz comparison of segmentations.

## When NOT to use

- The input is already an aggregated cell-by-gene count table — re-running aggregation would be redundant
- No cell segmentation exists yet — segmentation (e.g. Baysor on transcript patches) must precede aggregation
- The data are protein-staining-only (e.g. MACSima/PhenoCycler) without a transcript layer — gene counting does not apply

## Inputs

- SpatialData .zarr object from MERSCOPE or Xenium raw data
- Cell segmentation (vendor cell_id / cell_boundaries, or Sopa/Baysor merged polygons)
- Transcript dataframe with gene-name column (e.g. 'genes' for MERSCOPE)

## Outputs

- Cell-by-gene count matrix (AnnData table within the SpatialData object)
- Per-cell metadata after min-transcripts filtering (low-quality cells removed)
- Aggregated object suitable for clustering and UMAP rendering

## How to apply

Load the raw MERSCOPE or Xenium data into a SpatialData .zarr object (sopa.io.merscope / sopa.io.xenium), then run Sopa's aggregation step with the --aggregate-genes flag so transcripts are counted inside each cell polygon. Before counting, apply sopa.settings.gene_exclude_pattern ('negcontrol.*|blank.*|antisense.*|unassigned.*|deprecated.*|intergenic.*') to drop vendor control/blank gene names during both segmentation and aggregation. Apply a min-transcripts threshold to remove cells with low transcript counts, a decision point that trades off cell yield against count-matrix quality. The rationale is that averaged staining and counted transcripts inside each cell are what enable downstream tasks such as annotation, clustering, and cluster-separation comparison across segmentations.

## Related tools

- **Sopa** (Provides the aggregation command (sopa aggregate --aggregate-genes) that counts transcripts inside each cell and applies min-transcripts filtering) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Unified data framework holding the .zarr object with transcripts, shapes, and the aggregated AnnData table) — https://github.com/scverse/spatialdata

## Examples

```
sopa aggregate . --aggregate-genes
```

## Evaluation signals

- Aggregation completed without leaving low-quality cells: verify a min-transcripts threshold was applied to the aggregated table
- Control/blank gene columns absent from the final var index (gene_exclude_pattern 'negcontrol.*|blank.*|antisense.*|unassigned.*|deprecated.*|intergenic.*' took effect)
- Count matrix cell count matches the resolved segmentation (no unassigned '-1'/'UNASSIGNED' cells leaking through)
- Downstream clustering on the aggregated matrix yields UMAP plots and cluster-separation metrics (Calinski-Harabasz, mean cluster distance) comparable to or better than vendor segmentation as in Fig. 3a-f
- Validation file fig3_segmentation_comparison_metrics.csv exists with all three metric columns for both datasets and both segmentations

## Limitations

- Baysor's min_molecules_per_cell=10 and the min-transcripts aggregation threshold both discard cells with very few transcripts; boundaries for such cells are unreliable
- Vendor-prior transcripts with unassigned values (MERSCOPE cell_id=-1, Xenium 'UNASSIGNED') must be handled via prior_shapes_key or they will not aggregate correctly
- The gene_exclude_pattern must be set before aggregation; control/blank genes otherwise contaminate the count matrix

## Evidence

- [intro] transcript counting with downstream annotation: "we average the staining intensities and count the transcripts inside each cell, allowing further tasks such as annotation."
- [methods] gene_exclude_pattern filter: "Use `sopa.settings.gene_exclude_pattern` to filter out gene names during segmentation and aggregation."
- [methods] min_transcripts cell filter: "A [min_transcripts](/api/aggregation/#sopa.aggregate) threshold to remove cells with a low transcript count"
