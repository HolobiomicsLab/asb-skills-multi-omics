---
name: transcript-based-cell-segmentation
description: Use when processing MERSCOPE or Xenium transcript data where vendor segmentation
  is unavailable, unsatisfying, or higher-resolution cell-type distinction is needed
  and cell boundaries must be inferred from transcript composition.
when_to_use_negative:
- Input is protein-staining imaging data (MACSima, PhenoCycler) — use image-based
  segmentation (Cellpose) instead, since Baysor works on transcript positions
- A prior image-based segmentation already missed many cells — Baysor relies on the
  prior to determine cell number/location and cannot introduce new cells
- Data are already segmented and an aggregated cell-by-gene table is all that is needed
  — skip to annotation/aggregation steps
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_6398
tools:
- name: Sopa
  role: Pipeline that patchifies transcripts, runs Baysor per patch, resolves merges,
    and aggregates gene counts; install with the baysor extra (pip install 'sopa[baysor]')
  repo: https://github.com/gustaveroussy/sopa
- name: Baysor
  role: Transcript-based segmentation engine run on each patch using spatial position,
    local gene composition, and optional prior segmentation masks
  repo: https://github.com/kharchenkolab/Baysor
- name: SpatialData
  role: Unified data framework holding the converted .zarr objects on which segmentation
    and aggregation operate
  repo: https://github.com/scverse/spatialdata
- name: Cellpose
  role: Optional image-based segmentation whose masks can serve as the prior for transcript-based
    segmentation
  repo: https://github.com/gustaveroussy/sopa
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:1a010d7848782b92d45bdb644174ba457f2c9d4a9167ca246141a2cb2cb41003
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_6398
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# transcript-based-cell-segmentation

## Summary

Segments cells in imaging-based spatial transcriptomics (MERSCOPE, Xenium) directly from transcript positions using Baysor within the Sopa pipeline, run per patch and merged, optionally using vendor or Cellpose segmentation as a prior.

## When to use

Use when processing MERSCOPE or Xenium transcript data where vendor segmentation is unavailable, unsatisfying, or higher-resolution cell-type distinction is needed and cell boundaries must be inferred from transcript composition.

## When NOT to use

- Input is protein-staining imaging data (MACSima, PhenoCycler) — use image-based segmentation (Cellpose) instead, since Baysor works on transcript positions
- A prior image-based segmentation already missed many cells — Baysor relies on the prior to determine cell number/location and cannot introduce new cells
- Data are already segmented and an aggregated cell-by-gene table is all that is needed — skip to annotation/aggregation steps

## Inputs

- MERSCOPE raw transcript/cell data (prior_shapes_key='cell_id', unassigned_value=-1)
- Xenium raw transcript data (prior_shapes_key='UNASSIGNED')
- SpatialData .zarr object produced by sopa convert
- Baysor TOML config (force_2d=true, min_molecules_per_cell=10, scale=6, scale_std='25%')

## Outputs

- Per-cell polygon segmentation merged across patches
- Cell-by-gene count matrix (aggregate --aggregate-genes)
- fig3_segmentation_comparison_metrics.csv with Calinski-Harabasz, mean cluster distance, and DE gene score columns
- UMAP plots colored by cell type

## How to apply

Convert raw MERSCOPE/Xenium data into a SpatialData .zarr object with sopa.io.merscope or sopa.io.xenium, then apply the default gene filtering via sopa.settings.gene_exclude_pattern ('negcontrol.*|blank.*|antisense.*|unassigned.*|deprecated.*|intergenic.*'). Patchify the transcripts (e.g. --patch-width-microns 200) and supply the vendor segmentation as a prior via prior_shapes_key ('cell_id' with unassigned_value=-1 for MERSCOPE, 'UNASSIGNED' for Xenium). Run Baysor on each transcript patch with a TOML config (force_2d=true, min_molecules_per_cell=10, scale=6 microns, scale_std='25%'), then merge patch segmentations with 'sopa resolve baysor --gene-column genes' to remove cross-patch artefacts. Aggregate per-cell gene counts with 'sopa aggregate --aggregate-genes', applying a min-transcripts threshold to remove low-quality cells. Correctness is judged by comparing cluster separation against the proprietary Vizgen/10X segmentations using the Calinski-Harabasz score, mean cluster distance, and differential-expression gene score on UMAP plots.

## Related tools

- **Sopa** (Pipeline that patchifies transcripts, runs Baysor per patch, resolves merges, and aggregates gene counts; install with the baysor extra (pip install 'sopa[baysor]')) — https://github.com/gustaveroussy/sopa
- **Baysor** (Transcript-based segmentation engine run on each patch using spatial position, local gene composition, and optional prior segmentation masks) — https://github.com/kharchenkolab/Baysor
- **SpatialData** (Unified data framework holding the converted .zarr objects on which segmentation and aggregation operate) — https://github.com/scverse/spatialdata
- **Cellpose** (Optional image-based segmentation whose masks can serve as the prior for transcript-based segmentation) — https://github.com/gustaveroussy/sopa

## Examples

```
sopa convert --technology merscope && sopa resolve baysor --gene-column genes && sopa aggregate --aggregate-genes
```

## Evaluation signals

- fig3_segmentation_comparison_metrics.csv exists with all three metric columns (Calinski-Harabasz score, mean cluster distance, differential-expression gene score) for both datasets (MERSCOPE HCC liver, Xenium pancreatic cancer) and both segmentations
- UMAP figures are generated and Sopa/Baysor shows improved cell-type distinction and better cluster separation relative to the Vizgen and 10X Genomics vendor segmentations, as in Fig. 3a-f
- Prior shapes are correctly wired: 'cell_id' with unassigned_value=-1 for MERSCOPE, 'UNASSIGNED' for Xenium; negative-control/blank genes matching gene_exclude_pattern are excluded before segmentation and aggregation
- Low-quality cells are removed after aggregation by the min-transcripts threshold

## Limitations

- Cells with very few observed transcripts will tend to have unreliable boundaries
- Proseg/Baysor-style transcript segmentation relies on a prior segmentation to determine the number and approximate location of cells and cannot introduce new cells if the prior missed them
- In highly homogeneous regions of cells with very similar expression profiles, boundaries between cells will be much less reliable
- Future segmentation/annotation methods can be added to Sopa only once published and validated

## Evidence

- [methods] Baysor per-patch run with prior: [evidence span withheld — anchor status: not_found]
- [intro] Segmentation engines supported by Sopa: "Segmentation can then be performed for each individual patch, and we currently support Cellpose18 (image-based segmentation) and Baysor19 (transcripts-based segmentation)."
- [methods] Gene exclusion pattern applied during segmentation and aggregation: "Use `sopa.settings.gene_exclude_pattern` to filter out gene names during segmentation and aggregation."
- [readme] Baysor uses transcript position, gene composition, and prior masks: [evidence span withheld — anchor status: not_found]
- [results] Improved cluster separation vs proprietary segmentation: [evidence span withheld — anchor status: not_found]
- [intro] Merging masks across patches removes artefacts: "the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts."
