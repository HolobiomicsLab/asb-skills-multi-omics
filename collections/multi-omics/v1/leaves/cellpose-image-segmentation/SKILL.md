---
name: cellpose-image-segmentation
description: Use when segmenting cells from large staining images (Xenium, MERSCOPE,
  CosMX, MACSima, PhenoCycler) in a SpatialData object, especially when whole-image
  Cellpose would exhaust memory on millions of cells.
when_to_use_negative:
- The goal is transcript-based segmentation without images — use Baysor (or Proseg)
  on the transcript dataframe instead
- The input is already a segmented cell table (e.g. pre-computed boundaries); Sopa's
  segmentation steps assume raw images or transcripts
- Images are small enough that whole-image Cellpose fits comfortably in RAM — patching
  adds pipeline overhead without benefit
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: Sopa
  role: Pipeline that patchifies the image, runs Cellpose per patch, resolves patch-boundary
    conflicts, and aggregates results; installed via 'uv sync --all-extras --dev'
  repo: github.com/gustaveroussy/sopa
- name: Cellpose
  role: Image-based segmentation model executed on each overlapping image patch
- name: SpatialData
  role: Unified data framework holding the image, patches, and segmentation outputs
  repo: github.com/scverse/spatialdata
- name: Xenium
  role: Example imaging technology whose image crops are segmented with Cellpose
- name: MERSCOPE
  role: Example imaging technology whose image crops are segmented with Cellpose
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:13272c25e4c74bd5a718b084db61b00cfa4f9b1fec8d847b57e566d47c5803a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - Cellpose
  - MERSCOPE
  - Sopa
  - SpatialData
  - Xenium
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# cellpose-image-segmentation

## Summary

Run Cellpose nuclei/cell segmentation on image-based spatial omics data, either on the whole image or, preferably via Sopa, on overlapping image patches (e.g. 1500 px wide with 50 px overlap) that drastically reduce RAM and runtime before merging patch-level masks into cell polygons.

## When to use

Use when segmenting cells from large staining images (Xenium, MERSCOPE, CosMX, MACSima, PhenoCycler) in a SpatialData object, especially when whole-image Cellpose would exhaust memory on millions of cells.

## When NOT to use

- The goal is transcript-based segmentation without images — use Baysor (or Proseg) on the transcript dataframe instead
- The input is already a segmented cell table (e.g. pre-computed boundaries); Sopa's segmentation steps assume raw images or transcripts
- Images are small enough that whole-image Cellpose fits comfortably in RAM — patching adds pipeline overhead without benefit

## Inputs

- Staining image (e.g. DAPI/nuclear channel) from Xenium, MERSCOPE, CosMX, MACSima, or PhenoCycler, loaded as a SpatialData object via sopa.io readers
- Optional tissue/ROI selection excluding low-quality areas
- Segmentation parameters (diameter, min_area) supplied to the Cellpose call

## Outputs

- Per-patch Cellpose segmentation masks
- Merged cell boundary polygons after 'sopa resolve cellpose' conflict resolution
- Per-cell aggregated staining intensities (after the aggregation step)

## How to apply

Load the image into a SpatialData object via a Sopa reader (e.g. sopa.io.xenium, sopa.io.merscope), optionally restricting to the tissue with sopa.segmentation.tissue. Instead of segmenting the whole image, generate overlapping patches (e.g. 'sopa patchify image --patch-width-pixel 1500 --patch-overlap-pixel 50') and run Cellpose per patch with parameters such as diameter 35 and min_area 2000 to remove small artefacts. Rationale: per-patch segmentation needs an order of magnitude less memory (10-100x less RAM, up to 100x faster) than whole-image runs. Afterwards, run 'sopa resolve cellpose' to convert masks into polygons and merge them across patches, removing boundary artefacts; optionally combine with transcript-based Baysor via sopa.segmentation.combine and aggregate staining intensities and transcripts per cell.

## Related tools

- **Sopa** (Pipeline that patchifies the image, runs Cellpose per patch, resolves patch-boundary conflicts, and aggregates results; installed via 'uv sync --all-extras --dev') — github.com/gustaveroussy/sopa
- **Cellpose** (Image-based segmentation model executed on each overlapping image patch)
- **SpatialData** (Unified data framework holding the image, patches, and segmentation outputs) — github.com/scverse/spatialdata
- **Xenium** (Example imaging technology whose image crops are segmented with Cellpose)
- **MERSCOPE** (Example imaging technology whose image crops are segmented with Cellpose)

## Examples

```
sopa patchify image ... --patch-width-pixel 1500 --patch-overlap-pixel 50 && sopa resolve cellpose
```

## Evaluation signals

- Patched Cellpose runs show substantially lower peak RAM than the whole-image baseline (order-of-magnitude, 10-100x memory savings)
- Patched runtime is up to ~100x faster than the unpatched whole-image run
- No spurious cells at patch boundaries after 'sopa resolve cellpose' (overlap of 50 px plus polygon merging removes artefacts)
- Small objects below min_area 2000 are filtered from the segmentation
- benchmark_results.csv contains memory-ratio and time-ratio entries for Cellpose per dataset, with patched ratios << 1

## Limitations

- Cells straddling patch boundaries depend on the overlap and polygon-merge step; too-small overlaps can leave segmentation artefacts
- Cellpose (image-based) cannot segment cells absent from the staining image; transcript-only data require Baysor
- Whole-image Cellpose on millions of cells demands high-performance clusters with substantial memory; the paper notes most existing methods are 'not designed to handle large images with millions of cells'

## Evidence

- [intro] Segmentation can then be performed for each individual patch, and we currently support Cellpose18 (image-based segmentation) and Baysor19 (transcripts-based segmentation).: "Segmentation can then be performed for each individual patch, and we currently support Cellpose18 (image-based segmentation) and Baysor19 (transcripts-based segmentation)."
- [intro] the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts: "the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts"
- [methods] Using a [min_area](../api/segmentation/#sopa.segmentation.cellpose) threshold to remove small cells (provide this argument to the segmentation methods).: "Using a [min_area](../api/segmentation/#sopa.segmentation.cellpose) threshold to remove small cells (provide this argument to the segmentation methods)."
- [intro] Most existing methods18–20 are not designed to handle large images with millions of cells. Their usage typically demands high-performance computational clusters with substantial memory resources: "Most existing methods18–20 are not designed to handle large images with millions of cells. Their usage typically demands high-performance computational clusters with substantial memory"
- [other] Run the baseline (no patching) segmentations: Cellpose on the whole image and Baysor on the whole transcript set, measuring peak RAM and wall-clock time for each.: [evidence span withheld — anchor status: not_found]
