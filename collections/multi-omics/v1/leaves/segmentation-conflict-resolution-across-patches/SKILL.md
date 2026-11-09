---
name: segmentation-conflict-resolution-across-patches
description: Use when large images must be segmented patch-wise (e.g., memory limits)
  and you need to verify that overlapping patches do not introduce boundary conflicts
  between duplicated cell segmentations.
when_to_use_negative:
- The image is small enough to segment in full without patching — running full-image
  segmentation avoids boundary conflicts entirely.
- Segmentation is transcript-based (e.g., Baysor) rather than image-based — the IOMA
  boundary-conflict analysis applies to image-derived cell masks converted to polygons.
- You need to detect partially conflicting cells with intermediate IOMA scores (~0.07–0.8);
  by design these are the rare residual artefacts this workflow does not resolve.
edam_operation: http://edamontology.org/operation_3552
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: Sopa
  role: Pipeline that generates overlapping patches, runs per-patch segmentation,
    and merges masks into polygons to resolve conflicts (`sopa patchify`, `sopa resolve`)
  repo: https://github.com/gustaveroussy/sopa
- name: Cellpose
  role: Image-based segmentation method run on each patch (e.g., --channels DAPI --diameter
    35 --min-area 2000) before mask merging
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Unified data framework storing the converted dataset, patches, and merged
    cell polygons as a .zarr SpatialData object
  repo: https://github.com/scverse/spatialdata
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:cae33fbbce097f7b5bb5f974d407915df638bfc186e5bbf562136d27d292e95b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - Cellpose
  - Sopa
  - SpatialData
  edam_operation: http://edamontology.org/operation_3552
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# segmentation-conflict-resolution-across-patches

## Summary

A Sopa workflow that segments image-based spatial omics data on overlapping patches, merges per-patch Cellpose masks into polygons, and quantifies boundary conflicts using the intersection-over-min-area (IOMA) metric to verify that patch-based segmentation does not compromise segmentation quality.

## When to use

Use when large images must be segmented patch-wise (e.g., memory limits) and you need to verify that overlapping patches do not introduce boundary conflicts between duplicated cell segmentations.

## When NOT to use

- The image is small enough to segment in full without patching — running full-image segmentation avoids boundary conflicts entirely.
- Segmentation is transcript-based (e.g., Baysor) rather than image-based — the IOMA boundary-conflict analysis applies to image-derived cell masks converted to polygons.
- You need to detect partially conflicting cells with intermediate IOMA scores (~0.07–0.8); by design these are the rare residual artefacts this workflow does not resolve.

## Inputs

- Image-based spatial omics data convertible to a SpatialData object (.zarr store, e.g., via `sopa convert`)
- Image channels used for segmentation (e.g., DAPI staining channel)
- Overlapping image patches (e.g., width 1500 px, 50 px overlap)

## Outputs

- Merged cell polygon boundaries resolved across patches (SpatialData shapes)
- ioma_scores.csv containing intersection-over-min-area conflict scores for duplicated cell boundaries
- ioma_conflict_distribution.png histogram of IOMA scores

## How to apply

Convert the input data into a SpatialData object (e.g., `sopa convert . --sdata-path tuto.zarr --technology toy_dataset`), then generate overlapping image patches with `sopa patchify image` (e.g., --patch-width-pixel 1500 --patch-overlap-pixel 50, yielding 4 patches on the toy dataset). Run Cellpose segmentation independently on each patch (e.g., `sopa segmentation cellpose --channels DAPI --diameter 35 --min-area 2000`), then merge the per-patch masks into polygons with `sopa resolve cellpose` to remove artefacts at patch boundaries. The rationale is that most conflicting cell boundaries across adjacent patches have an IOMA lower than 0.07 (same cell segmented twice) or higher than 0.8 (distinct cells slightly overlapping), so conflicts resolve cleanly. For validation, compute the IOMA conflict score for each pair of duplicated cell boundaries across adjacent patches, export to ioma_scores.csv, and plot the histogram (ioma_conflict_distribution.png), confirming the bimodal pattern below 0.07 or above 0.8 with all scores within [0, 1].

## Related tools

- **Sopa** (Pipeline that generates overlapping patches, runs per-patch segmentation, and merges masks into polygons to resolve conflicts (`sopa patchify`, `sopa resolve`)) — https://github.com/gustaveroussy/sopa
- **Cellpose** (Image-based segmentation method run on each patch (e.g., --channels DAPI --diameter 35 --min-area 2000) before mask merging) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Unified data framework storing the converted dataset, patches, and merged cell polygons as a .zarr SpatialData object) — https://github.com/scverse/spatialdata

## Examples

```
sopa patchify image tuto.zarr --patch-width-pixel 1500 --patch-overlap-pixel 50; sopa segmentation cellpose tuto.zarr --channels DAPI --diameter 35 --min-area 2000; sopa resolve cellpose tuto.zarr
```

## Evaluation signals

- IOMA score distribution is bimodal: most conflicts fall below 0.07 (same cell segmented twice) or above 0.8 (distinct cells slightly overlapping), reproducing Fig. 2e / Supplementary Table 1
- All IOMA scores lie within [0, 1]
- ioma_scores.csv contains one conflict score per pair of duplicated cell boundaries across adjacent patches
- Patchify step yields the expected number of patches (e.g., 4 patches on the toy dataset with 1500 px width and 50 px overlap)
- Merged polygon boundaries contain no artefactual duplicate cells at patch seams after `sopa resolve`

## Limitations

- Conflict resolution relies on IOMA extremes; boundaries with intermediate overlap (between 0.07 and 0.8) represent residual conflicts that may need manual inspection.
- The reported validation was performed on a small generated toy dataset (`sopa.io.toy_dataset`); behavior on large real images with millions of cells may differ.
- Segmentation quality depends on Cellpose parameters (--diameter, --min-area); poor per-patch masks propagate into merged polygons.

## Evidence

- [results] Patch-based segmentation does not impact segmentation quality: [evidence span withheld — anchor status: not_found]
- [intro] Merging masks into polygons: "the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts."
- [intro] Overlapping patches: "we generate overlapping patches of images and/or transcripts."
- [intro] Cellpose per patch: "Segmentation can then be performed for each individual patch, and we currently support Cellpose18 (image-based segmentation) and Baysor19 (transcripts-based segmentation)."
- [intro] SpatialData conversion: "Sopa converts any data into a SpatialData object, on which all of the six following tasks are performed."
