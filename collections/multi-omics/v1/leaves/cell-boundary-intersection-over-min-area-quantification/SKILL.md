---
name: cell-boundary-intersection-over-min-area-quantification
description: Use when segmentation was run on overlapping image patches and you need
  to quantify whether per-patch masks conflict at patch seams before merging them
  into final cell polygons.
when_to_use_negative:
- Segmentation was run on full images rather than overlapping patches, so no duplicated
  boundaries exist to compare
- Transcript-based segmentation (e.g., Baysor) without image-derived cell masks, where
  area-based boundary overlap is not the relevant conflict measure
- You need to fix conflicts rather than quantify them — Sopa's own merging/resolution
  step (`sopa resolve`) handles artifact removal
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: Sopa
  role: Generates overlapping patches, runs and resolves per-patch segmentation, and
    converts masks to polygons whose conflicts are scored with IOMA
  repo: https://github.com/gustaveroussy/sopa
- name: Cellpose
  role: Performs image-based cell segmentation independently on each patch (channels
    DAPI, diameter 35, min-area 2000), producing the masks whose boundary conflicts
    are quantified
- name: SpatialData
  role: Unified data framework holding the converted toy dataset (.zarr), patches,
    and segmentation results
  repo: https://github.com/scverse/spatialdata
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:2c85b05f7ce29008e548da86ff04aaf0b9a198d0568ca6e27181dbf917c96fad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - Cellpose
  - Sopa
  - SpatialData
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# cell-boundary-intersection-over-min-area-quantification

## Summary

Quantifies segmentation conflicts introduced by patch-based segmentation by computing the intersection-over-min-area (IOMA) score for each pair of duplicated cell boundaries across adjacent patches, then inspecting the score distribution to decide whether patching compromises segmentation quality.

## When to use

Use when segmentation was run on overlapping image patches and you need to quantify whether per-patch masks conflict at patch seams before merging them into final cell polygons.

## When NOT to use

- Segmentation was run on full images rather than overlapping patches, so no duplicated boundaries exist to compare
- Transcript-based segmentation (e.g., Baysor) without image-derived cell masks, where area-based boundary overlap is not the relevant conflict measure
- You need to fix conflicts rather than quantify them — Sopa's own merging/resolution step (`sopa resolve`) handles artifact removal

## Inputs

- Per-patch cell segmentation masks from overlapping image patches (e.g., Cellpose masks stored in a SpatialData .zarr object)
- Patch geometry (patch width and overlap in pixels, e.g., 1500 px width, 50 px overlap) defining adjacent-patch overlap regions

## Outputs

- ioma_scores.csv — table of IOMA conflict scores for duplicated cell boundary pairs across adjacent patches
- ioma_conflict_distribution.png — histogram of IOMA scores showing the bimodal conflict pattern

## How to apply

First run segmentation independently on each overlapping patch (e.g., Cellpose on the DAPI channel with --diameter 35 --min-area 2000 after `sopa patchify image` with a fixed patch width and overlap, e.g., 1500 px width and 50 px overlap). Then merge per-patch masks into polygons (`sopa resolve cellpose`) and, for every pair of duplicated cell boundaries in the overlap region between adjacent patches, compute the intersection-over-min-area (IOMA) score, i.e., overlap area divided by the smaller cell's area, giving values in [0, 1]. Export all scores to a CSV (e.g., ioma_scores.csv) and plot a histogram (e.g., ioma_conflict_distribution.png). Interpret the distribution bimodally: conflicts with IOMA below ~0.07 are distinct cells slightly overlapping, while those above ~0.8 are the same cell segmented twice; if most conflicts fall in these two regimes, patch-based segmentation does not degrade segmentation quality.

## Related tools

- **Sopa** (Generates overlapping patches, runs and resolves per-patch segmentation, and converts masks to polygons whose conflicts are scored with IOMA) — https://github.com/gustaveroussy/sopa
- **Cellpose** (Performs image-based cell segmentation independently on each patch (channels DAPI, diameter 35, min-area 2000), producing the masks whose boundary conflicts are quantified)
- **SpatialData** (Unified data framework holding the converted toy dataset (.zarr), patches, and segmentation results) — https://github.com/scverse/spatialdata

## Examples

```
sopa patchify image tuto.zarr --patch-width-pixel 1500 --patch-overlap-pixel 50; sopa segmentation cellpose tuto.zarr --channels DAPI --diameter 35 --min-area 2000; sopa resolve cellpose tuto.zarr
```

## Evaluation signals

- All computed IOMA scores lie within [0, 1], as expected for intersection-over-min-area
- The histogram of IOMA scores reproduces the reported bimodal pattern: most conflicts below 0.07 or above 0.8 (Fig. 2e, Supplementary Table 1)
- The number of scored boundary pairs matches the number of duplicated cells found in the overlap zones of adjacent patches
- ioma_scores.csv exports one row per conflicting boundary pair with a finite, non-negative score
- Merged polygons after `sopa resolve` show no residual duplicate cells in overlap regions

## Limitations

- Conclusions are validated on a generated toy dataset (sopa.io.toy_dataset); behavior on real, dense tissue with irregular boundaries may shift the IOMA distribution
- IOMA classifies conflicts but does not itself correct them; resolution still relies on Sopa's polygon merging over patches
- Results depend on patch overlap size (e.g., 50 px) — too little overlap may miss duplicated cells at seams

## Evidence

- [other] Patch-based segmentation does not impact segmentation quality because most conflicting cell boundaries have an intersection-over-min-area (IOMA) lower than 0.07 or higher than 0.8: [evidence span withheld — anchor status: not_found]
- [intro] Sopa generates overlapping patches and runs per-patch segmentation: "we generate overlapping patches of images and/or transcripts. Segmentation can then be performed for each individual patch"
- [intro] Masks are converted to polygons and merged over patches to remove artifacts: "the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts."
- [methods] Tutorial uses a generated dataset: "For this tutorial, we use a generated dataset. You can expect a total runtime of a few minutes."
