---
name: umap-cell-type-distinction-visualization
description: Use when comparing cell-type distinction between alternative segmentations
  of the same spatial-omics dataset (e.g., Sopa/Baysor vs Vizgen MERSCOPE or 10X Xenium
  vendor segmentation) via UMAP plots and cluster-separation metrics.
when_to_use_negative:
- Input is raw transcript data that has not yet been segmented and aggregated into
  a per-cell count matrix — run Baysor segmentation and sopa aggregate first
- Only one segmentation exists for the dataset — the skill's core value is the head-to-head
  comparison of Sopa/Baysor versus vendor segmentation metrics
- The technology provides only protein staining (e.g., MACSima, PhenoCycler) without
  transcripts — Baysor transcript-based segmentation and gene-count clustering do
  not apply
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Sopa
  role: 'Orchestrates the pipeline: converts data to SpatialData .zarr, patches transcripts,
    runs Baysor, merges segmentations, aggregates per-cell gene counts, and produces
    the comparison metrics and UMAP plots'
  repo: https://github.com/gustaveroussy/sopa
- name: Baysor
  role: Transcript-based segmentation engine whose cell assignments are clustered
    to assess cell-type distinction against vendor segmentations
  repo: github.com/kharchenkolab/Baysor
- name: SpatialData
  role: Universal data framework in which the MERSCOPE and Xenium inputs and aggregated
    cell tables are represented
  repo: https://github.com/scverse/spatialdata
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:f41e2a4d4904cadd58cf8f79f4bfe917afb3b550ab2dc47306a90a2346cdadcb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - Baysor
  - MERSCOPE
  - Sopa
  - SpatialData
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# umap-cell-type-distinction-visualization

## Summary

Quantify and visualize how well a spatial-transcriptomics segmentation distinguishes cell types by clustering aggregated per-cell gene counts, rendering UMAP plots colored by cell type, and computing Calinski-Harabasz score, mean cluster distance, and differential-expression gene score for Sopa/Baysor versus proprietary vendor segmentations (Vizgen MERSCOPE, 10X Genomics Xenium).

## When to use

Use when comparing cell-type distinction between alternative segmentations of the same spatial-omics dataset (e.g., Sopa/Baysor vs Vizgen MERSCOPE or 10X Xenium vendor segmentation) via UMAP plots and cluster-separation metrics.

## When NOT to use

- Input is raw transcript data that has not yet been segmented and aggregated into a per-cell count matrix — run Baysor segmentation and sopa aggregate first
- Only one segmentation exists for the dataset — the skill's core value is the head-to-head comparison of Sopa/Baysor versus vendor segmentation metrics
- The technology provides only protein staining (e.g., MACSima, PhenoCycler) without transcripts — Baysor transcript-based segmentation and gene-count clustering do not apply

## Inputs

- MERSCOPE HCC liver and Xenium pancreatic cancer raw data converted to SpatialData .zarr objects
- Per-cell gene count matrices (cell-by-gene) aggregated from Sopa/Baysor segmentation and from the proprietary Vizgen/10X segmentations
- Transcript tables with gene column and vendor prior segmentation keys ('cell_id'/-1 for MERSCOPE, 'UNASSIGNED' for Xenium)

## Outputs

- fig3_segmentation_comparison_metrics.csv with Calinski-Harabasz score, mean cluster distance, and differential-expression gene score columns for both datasets and both segmentations
- UMAP figures colored by cell type for each segmentation

## How to apply

Convert raw MERSCOPE or Xenium data into SpatialData .zarr objects (sopa.io.merscope / sopa.io.xenium), filter genes via sopa.settings.gene_exclude_pattern ('negcontrol.*|blank.*|antisense.*|unassigned.*|deprecated.*|intergenic.*'), run Baysor per transcript patch (force_2d=true, min_molecules_per_cell=10, scale=6 microns, scale_std='25%') with the vendor segmentation as prior (prior_shapes_key 'cell_id' with unassigned_value=-1 for MERSCOPE, 'UNASSIGNED' for Xenium), and merge patches with 'sopa resolve baysor --gene-column genes'. Aggregate per-cell gene counts with 'sopa aggregate --aggregate-genes' applying a min-transcripts threshold to drop low-quality cells, then cluster the aggregated cells and compute the Calinski-Harabasz score, mean cluster distance, and differential-expression gene score for each segmentation, rendering UMAP plots colored by cell type. The rationale is that better segmentation yields more significant population-specific genes and greater intra-cluster distance, so the Sopa/Baysor metrics should be comparable to or better than the vendor segmentations as reported in Fig. 3a-f.

## Related tools

- **Sopa** (Orchestrates the pipeline: converts data to SpatialData .zarr, patches transcripts, runs Baysor, merges segmentations, aggregates per-cell gene counts, and produces the comparison metrics and UMAP plots) — https://github.com/gustaveroussy/sopa
- **Baysor** (Transcript-based segmentation engine whose cell assignments are clustered to assess cell-type distinction against vendor segmentations) — github.com/kharchenkolab/Baysor
- **SpatialData** (Universal data framework in which the MERSCOPE and Xenium inputs and aggregated cell tables are represented) — https://github.com/scverse/spatialdata

## Examples

```
sopa resolve baysor --gene-column genes
```

## Evaluation signals

- fig3_segmentation_comparison_metrics.csv exists with all three metric columns (Calinski-Harabasz score, mean cluster distance, differential-expression gene score) for both datasets and both segmentations
- UMAP figures colored by cell type are generated for each segmentation
- Sopa/Baysor metrics are comparable to or better than the vendor (Vizgen/10X) segmentations as reported in Fig. 3a-f
- More significant population-specific genes are recovered for the Sopa/Baysor segmentation
- Greater intra-cluster (mean cluster) distance is observed for the Sopa/Baysor segmentation

## Limitations

- Low-quality cells with few transcripts must be removed via the min_transcripts aggregation threshold, otherwise they distort clustering and UMAP separation
- Gene names must be filtered (gene_exclude_pattern) to exclude negcontrol/blank/antisense/unassigned/deprecated/intergenic probes before aggregation, or metrics will be contaminated
- Current spatial technologies provide either proteins or transcripts; extra information layers (e.g., protein staining needed to extract the B cell population in the Xenium data) may be unavailable, limiting cell-type distinction

## Evidence

- [results] Compared to the proprietary segmentations, Sopa (leveraging Baysor) shows improved cell-type distinction on UMAP plots and better cluster separation: [evidence span withheld — anchor status: not_found]
- [full_text] Cluster the aggregated cells and compute the Calinski-Harabasz score, mean cluster distance, and differential-expression gene score: [evidence span withheld — anchor status: not_found]
- [intro] Segmentation can then be performed for each individual patch, and we currently support Cellpose18 (image-based segmentation) and Baysor19 (transcripts-based segmentation).: "Segmentation can then be performed for each individual patch, and we currently support Cellpose18 (image-based segmentation) and Baysor19 (transcripts-based segmentation)."
- [methods] A min_transcripts threshold to remove cells with a low transcript count: "A [min_transcripts](/api/aggregation/#sopa.aggregate) threshold to remove cells with a low transcript count"
- [readme] Baysor segments imaging-based spatial transcriptomics data using spatial position, local gene composition, and optional prior segmentation masks.: [evidence span withheld — anchor status: not_found]
