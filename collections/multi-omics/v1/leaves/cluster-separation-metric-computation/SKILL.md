---
name: cluster-separation-metric-computation
description: Use when two or more cell segmentations of the same spatial-transcriptomics
  run must be compared for cell-type resolution, e.g. Sopa/Baysor vs proprietary MERSCOPE/Xenium
  segmentations, using quantitative clustering metrics.
when_to_use_negative:
- 'Input is raw transcripts without any cell segmentation: metrics require aggregated
  per-cell counts, so segmentation (and aggregation) must precede this skill.'
- Only one segmentation is available with no alternative (e.g. vendor) baseline to
  compare against — the metrics are comparative by design (Sopa/Baysor vs Vizgen/10X).
- Data are protein-staining multiplex images (MACSima, PhenoCycler) rather than transcript-based
  platforms; the workflow targets transcript-based segmentation from Xenium/MERSCOPE/CosMX.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3316
tools:
- name: Sopa
  role: Converts raw MERSCOPE/Xenium data into SpatialData .zarr, patchifies transcripts,
    aggregates per-cell gene counts with 'sopa aggregate --aggregate-genes', and supplies
    the clustering input for metric computation
  repo: https://github.com/gustaveroussy/sopa
- name: Baysor
  role: Transcript-based segmentation run per patch with force_2d=true, min_molecules_per_cell=10,
    scale=6 microns, scale_std='25%'; its segmentation is the one being evaluated
    against vendor baselines
  repo: https://github.com/kharchenkolab/Baysor
- name: SpatialData
  role: Universal framework holding the converted .zarr objects on which segmentation
    and aggregation are performed before clustering
  repo: https://github.com/scverse/spatialdata
- name: MERSCOPE
  role: Vizgen spatial-transcriptomics platform providing the HCC liver dataset and
    its proprietary segmentation used as the comparison baseline
- name: Xenium
  role: 10X Genomics spatial-transcriptomics platform providing the pancreatic cancer
    dataset and its proprietary segmentation used as the comparison baseline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:2c64b6ac8cdea220428b14071457c72d8c2c7df5260fd63cdd6b5e9fb747fca1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - Baysor
  - MERSCOPE
  - Sopa
  - SpatialData
  - Xenium
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cluster-separation-metric-computation

## Summary

Quantify how well cell-type clusters separate after segmentation of imaging-based spatial transcriptomics data (MERSCOPE HCC liver, Xenium pancreatic cancer) by computing Calinski-Harabasz scores, mean intra-cluster distances, and differential-expression gene scores on aggregated per-cell counts, and rendering UMAPs colored by cell type to compare Sopa/Baysor segmentation against vendor (Vizgen/10X) segmentations.

## When to use

Use when two or more cell segmentations of the same spatial-transcriptomics run must be compared for cell-type resolution, e.g. Sopa/Baysor vs proprietary MERSCOPE/Xenium segmentations, using quantitative clustering metrics.

## When NOT to use

- Input is raw transcripts without any cell segmentation: metrics require aggregated per-cell counts, so segmentation (and aggregation) must precede this skill.
- Only one segmentation is available with no alternative (e.g. vendor) baseline to compare against — the metrics are comparative by design (Sopa/Baysor vs Vizgen/10X).
- Data are protein-staining multiplex images (MACSima, PhenoCycler) rather than transcript-based platforms; the workflow targets transcript-based segmentation from Xenium/MERSCOPE/CosMX.

## Inputs

- SpatialData .zarr objects of MERSCOPE HCC liver and Xenium pancreatic cancer data (via sopa.io.merscope / sopa.io.xenium)
- Per-cell gene-count matrices from Sopa/Baysor segmentation and from vendor (Vizgen / 10X Genomics) segmentations
- Cell-type cluster labels from clustering the aggregated cells
- Vendor prior segmentation keys ('cell_id' with unassigned_value=-1 for MERSCOPE; 'UNASSIGNED' for Xenium)

## Outputs

- fig3_segmentation_comparison_metrics.csv with Calinski-Harabasz, mean cluster distance, and DE gene score columns for both datasets and both segmentations
- UMAP figures colored by cell type for each segmentation
- Per-cluster differential-expression gene lists with significance of population-specific genes

## How to apply

First aggregate per-cell gene counts for each segmentation being compared (e.g. with 'sopa aggregate --aggregate-genes' applying a min-transcripts threshold to drop low-quality cells), after clustering the aggregated cells. For each segmentation, compute the Calinski-Harabasz score, the mean cluster distance, and a differential-expression gene score, then render UMAP plots colored by cell type for visual comparison (as in Fig. 3a-f). Run Baysor per transcript patch with force_2d=true, min_molecules_per_cell=10, scale=6 microns, scale_std='25%', merging with 'sopa resolve baysor --gene-column genes', and supply the vendor segmentation as a prior via prior_shapes_key ('cell_id' with unassigned_value=-1 for MERSCOPE, 'UNASSIGNED' for Xenium). Rationale: Sopa/Baysor is expected to yield improved cell-type distinction and better cluster separation versus the proprietary Vizgen and 10X Genomics segmentations, so metrics should be comparable to or better than the vendor baselines.

## Related tools

- **Sopa** (Converts raw MERSCOPE/Xenium data into SpatialData .zarr, patchifies transcripts, aggregates per-cell gene counts with 'sopa aggregate --aggregate-genes', and supplies the clustering input for metric computation) — https://github.com/gustaveroussy/sopa
- **Baysor** (Transcript-based segmentation run per patch with force_2d=true, min_molecules_per_cell=10, scale=6 microns, scale_std='25%'; its segmentation is the one being evaluated against vendor baselines) — https://github.com/kharchenkolab/Baysor
- **SpatialData** (Universal framework holding the converted .zarr objects on which segmentation and aggregation are performed before clustering) — https://github.com/scverse/spatialdata
- **MERSCOPE** (Vizgen spatial-transcriptomics platform providing the HCC liver dataset and its proprietary segmentation used as the comparison baseline)
- **Xenium** (10X Genomics spatial-transcriptomics platform providing the pancreatic cancer dataset and its proprietary segmentation used as the comparison baseline)

## Examples

```
sopa resolve baysor --gene-column genes && sopa aggregate --aggregate-genes --min-transcripts 10
```

## Evaluation signals

- fig3_segmentation_comparison_metrics.csv exists and contains all three metric columns (Calinski-Harabasz score, mean cluster distance, differential-expression gene score) for both datasets (MERSCOPE liver, Xenium pancreatic cancer) and both segmentations
- UMAP figures colored by cell type are generated for each segmentation and show improved cell-type distinction for Sopa/Baysor versus vendor segmentation
- Sopa/Baysor metrics are comparable to or better than the vendor segmentations, reproducing Fig. 3a-f
- Low-quality cells are removed: the min-transcripts threshold applied during 'sopa aggregate' filtered cells with low transcript counts
- Differential expression yields more significant population-specific genes for the Sopa/Baysor segmentation than for the proprietary segmentations

## Limitations

- Baysor relies on a prior segmentation (vendor or Cellpose masks) to determine cell number and location; if the prior missed many cells, Baysor cannot correct for that error (analogous limitation documented for prior-seeded methods in the README).
- The comparison is platform-specific: conclusions are grounded on MERSCOPE HCC liver and Xenium pancreatic cancer datasets, and the absence of a protein-staining layer limited cell-type extraction (e.g. B cells in Xenium).
- Cells with very few observed transcripts have unreliable boundaries, so the min-transcripts filter must be applied before metric computation to avoid degrading cluster separation.

## Evidence

- [results] Compared to the proprietary segmentations, Sopa (leveraging Baysor) shows improved cell-type distinction on UMAP plots and better cluster separation, with more significant population-specific genes and greater intra-cluster distance (Fig. 3a-f).: [evidence span withheld — anchor status: not_found]
- [intro] Segmentation can then be performed for each individual patch, and we currently support Cellpose18 (image-based segmentation) and Baysor19 (transcripts-based segmentation).: "we currently support Cellpose18 (image-based segmentation) and Baysor19 (transcripts-based segmentation)"
- [methods] A [min_transcripts](/api/aggregation/#sopa.aggregate) threshold to remove cells with a low transcript count: "A [min_transcripts](/api/aggregation/#sopa.aggregate) threshold to remove cells with a low transcript count"
- [methods] Use `sopa.settings.gene_exclude_pattern` to filter out gene names during segmentation and aggregation.: "Use `sopa.settings.gene_exclude_pattern` to filter out gene names during segmentation and aggregation."
- [readme] Proseg relies on prior (usually image-based) segmentation to determine the number and approximate location of cells. It doesn't introduce new cells, so if the prior segmentation missed many cells, Proseg is not able to correct for that error.: [evidence span withheld — anchor status: not_found]
