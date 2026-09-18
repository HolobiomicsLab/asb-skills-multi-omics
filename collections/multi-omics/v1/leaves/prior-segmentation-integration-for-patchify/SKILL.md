---
name: prior-segmentation-integration-for-patchify
description: Use when re-segmenting MERSCOPE or Xenium spatial-transcriptomics data
  with Baysor inside Sopa and a vendor or Cellpose segmentation is available to pass
  as a prior via prior_shapes_key.
when_to_use_negative:
- No prior segmentation exists and none can be generated (e.g. no image channel for
  Cellpose and no vendor cell_id column) — Baysor prior-based refinement cannot be
  applied as described
- Input is not imaging-based spatial transcriptomics from a supported reader (MERSCOPE,
  Xenium, CosMX) — the vendor-prior keys and gene_exclude_pattern do not apply
- The goal is to evaluate the proprietary segmentation itself rather than refine it
  — use the vendor output directly instead of running Baysor on patches
tools:
- name: Sopa
  role: Orchestrates patchification, passes the prior segmentation via prior_shapes_key,
    resolves and merges Baysor patches, and aggregates gene counts
  repo: https://github.com/gustaveroussy/sopa
- name: Baysor
  role: Transcript-based segmentation engine run per patch, using spatial position,
    local gene composition, and the supplied prior segmentation masks
- name: Cellpose
  role: Optional image-based segmentation whose masks can serve as the prior for Baysor
- name: SpatialData
  role: Unified .zarr data framework holding transcripts, prior shapes, and the refined
    segmentation
  repo: https://github.com/scverse/spatialdata
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:17f844da9927aa2544d09b5e9d1387b686b98c8083ede9e0b94c1670132bf8db
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2222222222222222
n_claims: 9
---

# prior-segmentation-integration-for-patchify

## Summary

A Sopa/Baysor workflow that supplies an existing vendor (Vizgen MERSCOPE, 10X Xenium) or Cellpose segmentation as a prior when running transcript-based Baysor segmentation over overlapping patches, producing a refined segmentation whose clustering quality can be benchmarked against the proprietary segmentation.

## When to use

Use when re-segmenting MERSCOPE or Xenium spatial-transcriptomics data with Baysor inside Sopa and a vendor or Cellpose segmentation is available to pass as a prior via prior_shapes_key.

## When NOT to use

- No prior segmentation exists and none can be generated (e.g. no image channel for Cellpose and no vendor cell_id column) — Baysor prior-based refinement cannot be applied as described
- Input is not imaging-based spatial transcriptomics from a supported reader (MERSCOPE, Xenium, CosMX) — the vendor-prior keys and gene_exclude_pattern do not apply
- The goal is to evaluate the proprietary segmentation itself rather than refine it — use the vendor output directly instead of running Baysor on patches

## Inputs

- MERSCOPE HCC liver raw data (readable by sopa.io.merscope, prior column 'cell_id' with unassigned_value=-1)
- Xenium pancreatic cancer raw data (readable by sopa.io.xenium, prior key 'UNASSIGNED')
- Vendor segmentation output (Vizgen or 10X Genomics cell assignments) or Cellpose masks to use as prior_shapes_key
- Baysor TOML config (force_2d=true, min_molecules_per_cell=10, scale=6 microns, scale_std='25%')

## Outputs

- Refined per-cell segmentation (polygons merged across patches) in the SpatialData .zarr object
- Aggregated cell-by-gene count matrix from 'sopa aggregate --aggregate-genes'
- Clustering metrics (Calinski-Harabasz score, mean cluster distance, DE gene score) e.g. in fig3_segmentation_comparison_metrics.csv
- UMAP plots colored by cell type comparing Sopa/Baysor vs vendor segmentation

## How to apply

Convert the raw data into a SpatialData .zarr object with sopa.io.merscope or sopa.io.xenium, then patchify the transcripts (e.g. --patch-width-microns 200) so Baysor can run per patch. Supply the vendor or Cellpose segmentation as a prior through prior_shapes_key — use 'cell_id' with unassigned_value=-1 for MERSCOPE and 'UNASSIGNED' for Xenium — so Baysor leverages spatial position, local gene composition, and the prior masks. Configure Baysor with a TOML file (force_2d=true, min_molecules_per_cell=10, scale=6 microns, scale_std='25%') and let Sopa's gene_exclude_pattern ('negcontrol.*|blank.*|antisense.*|unassigned.*|deprecated.*|intergenic.*') exclude vendor control genes during segmentation and aggregation. Merge per-patch segmentations with 'sopa resolve baysor --gene-column genes', which converts masks to polygons and removes cross-patch artefacts, then aggregate per-cell gene counts with 'sopa aggregate --aggregate-genes' and a min-transcripts threshold to drop low-quality cells. The rationale is that the prior anchors the number and approximate location of cells while Baysor refines boundaries using transcript composition, yielding better cell-type distinction than the vendor segmentation alone.

## Related tools

- **Sopa** (Orchestrates patchification, passes the prior segmentation via prior_shapes_key, resolves and merges Baysor patches, and aggregates gene counts) — https://github.com/gustaveroussy/sopa
- **Baysor** (Transcript-based segmentation engine run per patch, using spatial position, local gene composition, and the supplied prior segmentation masks)
- **Cellpose** (Optional image-based segmentation whose masks can serve as the prior for Baysor)
- **SpatialData** (Unified .zarr data framework holding transcripts, prior shapes, and the refined segmentation) — https://github.com/scverse/spatialdata

## Examples

```
sopa resolve baysor --gene-column genes
```

## Evaluation signals

- fig3_segmentation_comparison_metrics.csv exists with all three metric columns (Calinski-Harabasz score, mean cluster distance, DE gene score) for both MERSCOPE and Xenium datasets and both segmentations
- UMAP figures colored by cell type are generated for Sopa/Baysor and the vendor segmentations
- Accepted result: Sopa/Baysor metrics are comparable to or better than the vendor segmentations, as reported in Fig. 3a-f
- Prior integration is honored: unassigned transcripts are handled with the technology-specific sentinel (cell_id=-1 for MERSCOPE, 'UNASSIGNED' for Xenium)
- Patch merging produces no cross-patch artefacts after 'sopa resolve baysor', and low-quality cells are removed by the min-transcripts threshold

## Limitations

- Requires the Baysor command-line tool installed independently in addition to the 'sopa[baysor]' extra (e.g. uv sync --all-extras --dev)
- Cells with very few molecules give unreliable boundaries; Baysor's min_molecules_per_cell=10 and Sopa's min-transcripts aggregation threshold are needed to filter them
- Future segmentation or annotation methods can only be added to Sopa once published and validated; vendor-prior column names and unassigned sentinels differ per technology (MERSCOPE vs Xenium)

## Evidence

- [intro] Baysor per-patch with prior masks: "Segmentation can then be performed for each individual patch, and we currently support Cellpose18 (image-based segmentation) and Baysor19 (transcripts-based segmentation)."
- [readme] Prior segmentation masks used by Baysor: [evidence span withheld — anchor status: not_found]
- [intro] Cross-patch merge removes artefacts: "the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts."
- [methods] Gene exclude pattern applies to segmentation and aggregation: "Use `sopa.settings.gene_exclude_pattern` to filter out gene names during segmentation and aggregation."
