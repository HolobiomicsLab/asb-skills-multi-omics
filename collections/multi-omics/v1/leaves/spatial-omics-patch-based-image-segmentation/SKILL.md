---
name: spatial-omics-patch-based-image-segmentation
description: Use when a spatial-omics image (Xenium, MERSCOPE, CosMX, MACSima, PhenoCycler)
  is too large for whole-image segmentation and you need cell masks with patch-boundary
  conflicts resolved.
when_to_use_negative:
- Input data are already segmented (Sopa analysis libraries downstream expect cell
  polygons/tables, not raw images)
- The image is small enough that full-image segmentation is feasible and patch-boundary
  conflict resolution is unnecessary overhead
- Transcript-based segmentation without images is the goal and no Cellpose-compatible
  channel exists — Baysor (transcript-based) should be chosen instead
tools:
- name: Sopa
  role: Pipeline orchestrating patchification, per-patch segmentation, polygon conversion,
    and conflict-resolving merge
  repo: https://github.com/gustaveroussy/sopa
- name: Cellpose
  role: Image-based segmentation model run independently on each image patch
- name: SpatialData
  role: Universal data framework holding the converted images, patches, and segmentation
    outputs
  repo: https://github.com/scverse/spatialdata
- name: Baysor
  role: Alternative transcript-based segmentation backend supported for patch-level
    segmentation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:fc6cbce8792fdeb4bcca08beec83fb2b2b971625a3fa1a8f37ce6d5e8185a940
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - Baysor
  - Cellpose
  - Sopa
  - SpatialData
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spatial-omics-patch-based-image-segmentation

## Summary

A memory-efficient patch-based segmentation strategy from Sopa: large spatial-omics images are split into overlapping patches, Cellpose (image-based) or Baysor (transcript-based) segmentation is run per patch, and per-patch masks are converted to polygons and merged across patches to remove boundary artefacts.

## When to use

Use when a spatial-omics image (Xenium, MERSCOPE, CosMX, MACSima, PhenoCycler) is too large for whole-image segmentation and you need cell masks with patch-boundary conflicts resolved.

## When NOT to use

- Input data are already segmented (Sopa analysis libraries downstream expect cell polygons/tables, not raw images)
- The image is small enough that full-image segmentation is feasible and patch-boundary conflict resolution is unnecessary overhead
- Transcript-based segmentation without images is the goal and no Cellpose-compatible channel exists — Baysor (transcript-based) should be chosen instead

## Inputs

- Raw image-based spatial omics data (Xenium, MERSCOPE, CosMX, MACSima, PhenoCycler outputs)
- SpatialData .zarr store (produced by `sopa convert`)
- Segmentation channels/stainings (e.g., DAPI) and Cellpose parameters (diameter, min_area)

## Outputs

- Overlapping image patches stored in the SpatialData object
- Per-patch Cellpose segmentation masks
- Merged cell polygons with patch-boundary conflicts resolved
- Aggregated per-cell tables (transcript counts and staining intensities) for downstream annotation

## How to apply

First convert the input into a SpatialData object (e.g., `sopa convert . --sdata-path tuto.zarr --technology toy_dataset`). Then generate overlapping image patches with `sopa patchify image` (on the Sopa toy dataset, --patch-width-pixel 1500 and --patch-overlap-pixel 50 yield 4 patches). Run Cellpose segmentation on each patch independently (e.g., `sopa segmentation cellpose tuto.zarr --channels DAPI --diameter 35 --min-area 2000`), where min_area removes small spurious cells. Finally merge the per-patch segmentation masks into polygons with `sopa resolve cellpose` to remove potential artefacts at patch boundaries; conflict severity between duplicated boundaries can be quantified with the intersection-over-min-area (IOMA) score. The rationale is that patch-based segmentation does not degrade quality because most conflicting boundaries have IOMA < 0.07 (same cell segmented twice) or > 0.8 (distinct cells slightly overlapping).

## Related tools

- **Sopa** (Pipeline orchestrating patchification, per-patch segmentation, polygon conversion, and conflict-resolving merge) — https://github.com/gustaveroussy/sopa
- **Cellpose** (Image-based segmentation model run independently on each image patch)
- **SpatialData** (Universal data framework holding the converted images, patches, and segmentation outputs) — https://github.com/scverse/spatialdata
- **Baysor** (Alternative transcript-based segmentation backend supported for patch-level segmentation)

## Examples

```
sopa patchify image tuto.zarr --patch-width-pixel 1500 --patch-overlap-pixel 50 && sopa segmentation cellpose tuto.zarr --channels DAPI --diameter 35 --min-area 2000 && sopa resolve cellpose tuto.zarr
```

## Evaluation signals

- IOMA conflict scores of duplicated cell boundaries across adjacent patches fall mostly below 0.07 or above 0.8 (reproducing Fig. 2e / Supplementary Table 1)
- All IOMA scores lie within the valid range [0, 1]
- Patchification of the toy dataset with width 1500 px and overlap 50 px yields exactly 4 patches
- After `sopa resolve`, no duplicated cell polygons remain across patch overlaps (artefacts removed)
- Cells below the min_area threshold (e.g., 2000 in the Cellpose run) are absent from the merged output

## Limitations

- Sopa currently supports Cellpose (image-based) and Baysor (transcript-based) for per-patch segmentation; new methods can only be added once published and validated
- Patch-based segmentation introduces boundary conflicts that must be resolved by polygon merging; unmerged duplicates would corrupt cell counts
- Small spurious cells must be filtered via a min_area threshold at segmentation time, otherwise they persist into downstream aggregation

## Evidence

- [intro] Patch segmentation backend support: "Segmentation can then be performed for each individual patch, and we currently support Cellpose18 (image-based segmentation) and Baysor19 (transcripts-based segmentation)."
- [intro] Patch generation: "we generate overlapping patches of images and/or transcripts."
- [intro] Mask-to-polygon merge across patches: "the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts."
- [abstract] Sopa purpose: "We introduce Sopa ( https://github.com/gustaveroussy/sopa ), a technology-invariant, memory-efficient pipeline with a unified visualizer for all image-based spatial omics."
- [methods] min_area cell filtering: "Using a [min_area](../api/segmentation/#sopa.segmentation.cellpose) threshold to remove small cells (provide this argument to the segmentation methods)."
- [readme] Repository identity: "Spatial omics pipeline and analysis"
