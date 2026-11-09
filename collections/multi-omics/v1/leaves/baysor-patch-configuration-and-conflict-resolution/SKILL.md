---
name: baysor-patch-configuration-and-conflict-resolution
description: Use when segmenting imaging-based spatial transcriptomics data (e.g.
  MERSCOPE, Xenium) from transcript coordinates alone and the vendor segmentation
  is unsatisfactory or a refined, transcript-driven segmentation is desired.
when_to_use_negative:
- Input is already an aggregated cell-by-gene count table rather than transcript coordinates
  — segmentation has already happened.
- Image-based segmentation is the goal (e.g. H&E or staining channels) — use Cellpose
  instead, since Baysor is the transcript-based branch of Sopa's segmentation.
- The prior segmentation missed many cells and new cells must be introduced — prior-based
  refinement cannot recover cells that were never segmented.
edam_operation: http://edamontology.org/operation_3791
edam_topics: []
tools:
- name: Sopa
  role: Orchestrates patchification, per-patch Baysor execution, polygon conversion,
    and conflict-resolving merge of patch segmentations
  repo: https://github.com/gustaveroussy/sopa
- name: Baysor
  role: Performs transcript-based Bayesian cell segmentation on each transcript patch
    using spatial position, local gene composition, and optional prior segmentation
    masks
  repo: https://github.com/kharchenkolab/Baysor
- name: SpatialData
  role: Unified data framework holding the .zarr object with transcripts, prior shapes,
    and segmentation outputs
  repo: https://github.com/scverse/spatialdata
- name: Cellpose
  role: Optional image-based segmentation whose masks can be supplied as the prior_shapes_key
    input to Baysor refinement
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:2188578b3a538205dfe62d052ef0c681946a13c42cb9d1b5983a822e86568d9f
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
  edam_operation: http://edamontology.org/operation_3791
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# baysor-patch-configuration-and-conflict-resolution

## Summary

Run Baysor transcript-based segmentation in a patch-parallelized Sopa workflow: patchify MERSCOPE/Xenium transcripts, supply vendor or Cellpose segmentation as a prior, run Baysor per patch with a tuned TOML config, then merge patch segmentations with conflict resolution.

## When to use

Use when segmenting imaging-based spatial transcriptomics data (e.g. MERSCOPE, Xenium) from transcript coordinates alone and the vendor segmentation is unsatisfactory or a refined, transcript-driven segmentation is desired.

## When NOT to use

- Input is already an aggregated cell-by-gene count table rather than transcript coordinates — segmentation has already happened.
- Image-based segmentation is the goal (e.g. H&E or staining channels) — use Cellpose instead, since Baysor is the transcript-based branch of Sopa's segmentation.
- The prior segmentation missed many cells and new cells must be introduced — prior-based refinement cannot recover cells that were never segmented.

## Inputs

- MERSCOPE or Xenium raw spatial transcriptomics data (transcript coordinates with gene names)
- SpatialData .zarr object created via sopa.io.merscope / sopa.io.xenium
- Prior segmentation shapes (vendor 'cell_id' for MERSCOPE with unassigned_value=-1, or 'UNASSIGNED' column for Xenium)
- Baysor TOML configuration file (force_2d, min_molecules_per_cell, scale, scale_std)

## Outputs

- Per-patch Baysor cell segmentations
- Merged, conflict-resolved cell polygons in the SpatialData object
- Per-cell gene count matrix after 'sopa aggregate --aggregate-genes' with min-transcripts filtering

## How to apply

Install Sopa with the baysor extra and the Baysor command-line tool, then convert raw data into a SpatialData .zarr object via sopa.io.merscope or sopa.io.xenium. Apply the default gene filtering through sopa.settings.gene_exclude_pattern ('negcontrol.*|blank.*|antisense.*|unassigned.*|deprecated.*|intergenic.*') so control/blank probes never inform segmentation. Patchify the transcripts (e.g. --patch-width-microns 200) for memory-efficient parallelization, and pass the vendor segmentation as a prior via prior_shapes_key ('cell_id' with unassigned_value=-1 for MERSCOPE, 'UNASSIGNED' for Xenium). Run Baysor on each patch with a TOML config (force_2d=true, min_molecules_per_cell=10, scale=6 microns, scale_std='25%'), then merge patch outputs with 'sopa resolve baysor --gene-column genes', which converts cell masks to polygons and resolves cross-patch conflicts to remove artefacts; afterwards aggregate per-cell counts with a min-transcripts threshold.

## Related tools

- **Sopa** (Orchestrates patchification, per-patch Baysor execution, polygon conversion, and conflict-resolving merge of patch segmentations) — https://github.com/gustaveroussy/sopa
- **Baysor** (Performs transcript-based Bayesian cell segmentation on each transcript patch using spatial position, local gene composition, and optional prior segmentation masks) — https://github.com/kharchenkolab/Baysor
- **SpatialData** (Unified data framework holding the .zarr object with transcripts, prior shapes, and segmentation outputs) — https://github.com/scverse/spatialdata
- **Cellpose** (Optional image-based segmentation whose masks can be supplied as the prior_shapes_key input to Baysor refinement)

## Examples

```
sopa resolve baysor --gene-column genes
```

## Evaluation signals

- Verify fig3_segmentation_comparison_metrics.csv exists with all three metric columns (Calinski-Harabasz score, mean cluster distance, differential-expression gene score) for both datasets and both segmentations.
- Confirm UMAP plots colored by cell type are generated and Sopa/Baysor shows cluster separation comparable to or better than vendor segmentations as in Fig. 3a-f.
- Check that merged polygons show no cross-patch boundary artefacts after conflict resolution.
- Confirm low-transcript cells are removed after aggregation via the min-transcripts threshold.

## Limitations

- Baysor relies on the prior (vendor or image-based) segmentation to determine the number and approximate location of cells and cannot introduce new cells where the prior missed them.
- Cells with very few observed transcripts have unreliable boundaries; the min_molecules_per_cell=10 and aggregation min-transcripts thresholds are needed to remove low-quality cells.
- In highly homogeneous regions with similar expression profiles, boundaries between cells are much less reliable.
- Baysor must be installed independently as a command-line tool in addition to the sopa[baysor] extra.

## Evidence

- [methods] Patchify and supply prior: [evidence span withheld — anchor status: not_found]
- [methods] Baysor config and merge: [evidence span withheld — anchor status: not_found]
- [intro] Segmentation support: "Segmentation can then be performed for each individual patch, and we currently support Cellpose18 (image-based segmentation) and Baysor19 (transcripts-based segmentation)."
- [intro] Conflict resolution rationale: "the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts."
- [readme] Proseg limitation on priors: [evidence span withheld — anchor status: not_found]
- [readme] Baysor overview: [evidence span withheld — anchor status: not_found]
