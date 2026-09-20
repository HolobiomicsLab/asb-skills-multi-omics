---
name: cross-technology-segmentation-benchmarking
description: Decision support for choosing between open-source (Sopa/Baysor) and proprietary
  (Vizgen/10X Genomics) segmentation when raw imaging-based spatial transcriptomics
  data with transcripts plus a vendor segmentation prior is available and cell-type
  resolution must be compared.
when_to_use_negative:
- Image-only data with no transcript table — Baysor is transcripts-based, so this
  benchmarking path is inapplicable (use Cellpose image-based segmentation instead)
- Data where only a proprietary segmentation is available and no prior_shapes_key
  (e.g. cell_id/UNASSIGNED column) can be supplied to Baysor
- Analyses focused only on downstream annotation of already-segmented, already-benchmarked
  data — re-running the segmentation comparison adds no information
edam_operation: http://edamontology.org/operation_3432
tools:
- name: Sopa
  role: Technology-invariant pipeline that converts raw data to SpatialData, patchifies
    transcripts, runs and resolves Baysor, and aggregates per-cell counts for benchmarking
  repo: https://github.com/gustaveroussy/sopa
- name: Baysor
  role: Transcript-based segmentation engine whose output cell boundaries are compared
    against Vizgen and 10X Genomics vendor segmentations
  repo: https://github.com/kharchenkolab/Baysor
- name: SpatialData
  role: Universal framework underpinning the .zarr data representation used to hold
    transcripts, shapes, and aggregated tables for both technologies
  repo: https://github.com/scverse/spatialdata
- name: Cellpose
  role: Optional image-based segmentation whose masks can serve as a prior for Baysor
    when a vendor segmentation is not used
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:c2c8078bf6162f420a447c3ca5015ed2a219dd13dc2459af0efaa0f8cd933eb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - Baysor
  - Cellpose
  - MERSCOPE
  - Sopa
  - SpatialData
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# cross-technology-segmentation-benchmarking

## Summary

Benchmark open-source Sopa/Baysor transcript-based segmentation against proprietary vendor segmentations (Vizgen on MERSCOPE, 10X Genomics on Xenium) using quantitative clustering metrics. It quantifies whether Baysor's gene-composition-aware boundaries yield higher-resolution cell-type distinction than vendor masks.

## When to use

Decision support for choosing between open-source (Sopa/Baysor) and proprietary (Vizgen/10X Genomics) segmentation when raw imaging-based spatial transcriptomics data with transcripts plus a vendor segmentation prior is available and cell-type resolution must be compared.

## When NOT to use

- Image-only data with no transcript table — Baysor is transcripts-based, so this benchmarking path is inapplicable (use Cellpose image-based segmentation instead)
- Data where only a proprietary segmentation is available and no prior_shapes_key (e.g. cell_id/UNASSIGNED column) can be supplied to Baysor
- Analyses focused only on downstream annotation of already-segmented, already-benchmarked data — re-running the segmentation comparison adds no information

## Inputs

- Raw MERSCOPE HCC liver data (transcripts plus vendor 'cell_id' segmentation, unassigned_value=-1)
- Raw Xenium pancreatic cancer data (transcripts plus vendor 'UNASSIGNED' prior)
- SpatialData .zarr objects produced by sopa.io.merscope / sopa.io.xenium
- Baysor TOML config (force_2d=true, min_molecules_per_cell=10, scale=6 microns, scale_std='25%')

## Outputs

- fig3_segmentation_comparison_metrics.csv with Calinski-Harabasz score, mean cluster distance, and DE gene score columns for both datasets and both segmentations
- UMAP plots colored by cell type per segmentation
- Aggregated per-cell gene-count matrix (sopa aggregate --aggregate-genes)

## How to apply

Convert MERSCOPE and Xenium raw data into SpatialData .zarr objects with sopa.io.merscope / sopa.io.xenium (sopa convert --technology merscope / xenium). Patchify transcripts (e.g. --patch-width-microns 200) and supply the vendor segmentation as a prior via prior_shapes_key ('cell_id' with unassigned_value=-1 for MERSCOPE, 'UNASSIGNED' for Xenium), then run Baysor per patch with a TOML config (force_2d=true, min_molecules_per_cell=10, scale=6 microns, scale_std='25%') and merge with 'sopa resolve baysor --gene-column genes'. Filter gene names via sopa.settings.gene_exclude_pattern ('negcontrol.*|blank.*|antisense.*|unassigned.*|deprecated.*|intergenic.*') during segmentation and aggregation, then aggregate per-cell counts with 'sopa aggregate --aggregate-genes' applying a min-transcripts threshold to remove low-quality cells. Cluster the aggregated cells for both the Sopa/Baysor and vendor segmentations and compute the Calinski-Harabasz score, mean cluster distance, and differential-expression gene score, rendering UMAP plots colored by cell type. The rationale is that Baysor uses spatial position, local gene composition, and the prior masks, so boundaries should respect transcriptomic neighborhoods better than vendor polygon heuristics.

## Related tools

- **Sopa** (Technology-invariant pipeline that converts raw data to SpatialData, patchifies transcripts, runs and resolves Baysor, and aggregates per-cell counts for benchmarking) — https://github.com/gustaveroussy/sopa
- **Baysor** (Transcript-based segmentation engine whose output cell boundaries are compared against Vizgen and 10X Genomics vendor segmentations) — https://github.com/kharchenkolab/Baysor
- **SpatialData** (Universal framework underpinning the .zarr data representation used to hold transcripts, shapes, and aggregated tables for both technologies) — https://github.com/scverse/spatialdata
- **Cellpose** (Optional image-based segmentation whose masks can serve as a prior for Baysor when a vendor segmentation is not used)

## Examples

```
sopa resolve baysor --gene-column genes
```

## Evaluation signals

- fig3_segmentation_comparison_metrics.csv exists and contains all three metric columns (Calinski-Harabasz score, mean cluster distance, DE gene score) for both datasets and both segmentations
- UMAP figures colored by cell type are generated for each segmentation
- Accepted result: Sopa/Baysor metrics are comparable to or better than the vendor segmentations, as reported in Fig. 3a-f
- gene_exclude_pattern ('negcontrol.*|blank.*|antisense.*|unassigned.*|deprecated.*|intergenic.*') was applied so negative controls and blanks are excluded from segmentation and aggregation
- Low-quality cells were removed by the min-transcripts threshold in 'sopa aggregate --aggregate-genes'

## Limitations

- Baysor relies on a prior segmentation (vendor masks or Cellpose) to determine cell number and approximate location; it does not introduce new cells, so prior-segmentation errors propagate
- Baysor results are non-deterministic run to run, so repeated benchmarking runs will vary slightly
- Cells with very few observed transcripts have unreliable boundaries, hence the min-transcripts aggregation threshold

## Evidence

- [results] Sopa/Baysor vs proprietary segmentation metrics: [evidence span withheld — anchor status: not_found]
- [readme] Baysor segmentation basis: [evidence span withheld — anchor status: not_found]
- [intro] Segmentation backends supported per patch: "Segmentation can then be performed for each individual patch, and we currently support Cellpose18 (image-based segmentation) and Baysor19 (transcripts-based segmentation)."
- [methods] Gene-name filtering setting: "Use `sopa.settings.gene_exclude_pattern` to filter out gene names during segmentation and aggregation."
- [methods] Min-transcripts cell filter: "A [min_transcripts](/api/aggregation/#sopa.aggregate) threshold to remove cells with a low transcript count"
- [abstract] Technology-invariant framing: "We introduce Sopa ( https://github.com/gustaveroussy/sopa ), a technology-invariant, memory-efficient pipeline with a unified visualizer for all image-based spatial omics."
