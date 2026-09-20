---
name: overlapping-patch-tiling-strategy
description: Use when segmenting large spatial omics images (e.g. Xenium, MERSCOPE,
  CosMX) with millions of cells where whole-image Cellpose or whole-transcript Baysor
  runs exhaust RAM or wall-clock time.
when_to_use_negative:
- The image or transcript set is small enough that whole-image Cellpose or whole-transcript
  Baysor runs comfortably within available memory — patching adds unnecessary complexity
- The data is already segmented (Sopa's downstream tasks like annotation and spatial
  analysis assume cell boundaries already exist)
- The technology is not image-based spatial omics (e.g. spot-based or sequencing-based
  platforms) since patching is designed for images and transcript coordinates
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: Sopa
  role: Provides the patchify/resolve CLI commands and API that implement the overlapping-patch
    strategy and merge patch boundaries
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Unified data framework into which inputs are converted; all patching and segmentation
    tasks operate on the SpatialData object
  repo: https://github.com/scverse/spatialdata
- name: Cellpose
  role: Image-based segmentation model run per image patch (diameter 35, min_area
    2000)
- name: Baysor
  role: Transcript-based segmentation model run per transcript patch (scale=6 microns,
    min_molecules_per_cell=10)
- name: MERSCOPE
  role: Spatial transcriptomics platform whose data is read via sopa.io.merscope and
    segmented with the patch strategy
- name: Xenium
  role: Spatial transcriptomics platform whose data is read via sopa.io.xenium and
    segmented with the patch strategy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:dc75ebaaf89093c204f61ff8f1add10bcc87717bafd8acad99caac587c6edbd2
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
  - Xenium
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# overlapping-patch-tiling-strategy

## Summary

A memory- and time-efficient tiling strategy from Sopa that segments large image-based spatial omics datasets by applying Cellpose or Baysor to small overlapping patches of images/transcripts instead of the whole image, then merging patch-level cell masks to remove boundary artefacts.

## When to use

Use when segmenting large spatial omics images (e.g. Xenium, MERSCOPE, CosMX) with millions of cells where whole-image Cellpose or whole-transcript Baysor runs exhaust RAM or wall-clock time.

## When NOT to use

- The image or transcript set is small enough that whole-image Cellpose or whole-transcript Baysor runs comfortably within available memory — patching adds unnecessary complexity
- The data is already segmented (Sopa's downstream tasks like annotation and spatial analysis assume cell boundaries already exist)
- The technology is not image-based spatial omics (e.g. spot-based or sequencing-based platforms) since patching is designed for images and transcript coordinates

## Inputs

- Image-based spatial omics data (Xenium, MERSCOPE, CosMX, MACSima, PhenoCycler) loaded as a SpatialData object
- Transcript dataframes for Baysor segmentation
- Baysor config.toml with scale and min_molecules_per_cell parameters

## Outputs

- Patch-level segmentation masks converted to polygons and merged into whole-image cell boundary polygons
- Resolved cell segmentation free of patch-boundary artefacts
- benchmark_results.csv recording peak RAM and time ratios (unpatched vs patched) per method and dataset

## How to apply

Convert the input data into a SpatialData object (e.g. via sopa.io.merscope or sopa.io.xenium), then generate overlapping patches with 'sopa patchify image ... --patch-width-pixel 1500 --patch-overlap-pixel 50' for image-based segmentation or 'sopa patchify transcripts ... --patch-width-microns 200' for transcript-based segmentation. Run Cellpose per image patch (diameter 35, min_area 2000) or Baysor per transcript patch (config.toml with scale=6 microns, min_molecules_per_cell=10). Then resolve conflicts at patch boundaries with 'sopa resolve cellpose' or 'sopa resolve baysor', since cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts. Benchmark peak RAM and wall-clock time against unpatched whole-image/whole-transcript baselines to confirm the expected order-of-magnitude savings.

## Related tools

- **Sopa** (Provides the patchify/resolve CLI commands and API that implement the overlapping-patch strategy and merge patch boundaries) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Unified data framework into which inputs are converted; all patching and segmentation tasks operate on the SpatialData object) — https://github.com/scverse/spatialdata
- **Cellpose** (Image-based segmentation model run per image patch (diameter 35, min_area 2000))
- **Baysor** (Transcript-based segmentation model run per transcript patch (scale=6 microns, min_molecules_per_cell=10))
- **MERSCOPE** (Spatial transcriptomics platform whose data is read via sopa.io.merscope and segmented with the patch strategy)
- **Xenium** (Spatial transcriptomics platform whose data is read via sopa.io.xenium and segmented with the patch strategy)

## Examples

```
sopa patchify image . --patch-width-pixel 1500 --patch-overlap-pixel 50 && sopa patchify transcripts . --patch-width-microns 200
```

## Evaluation signals

- benchmark_results.csv exists and contains memory-ratio and time-ratio entries for both Cellpose and Baysor on each dataset
- Patched runs show substantially lower peak RAM than unpatched baselines — expected order-of-magnitude memory savings (10-100x less memory)
- Patched runs are up to 100x faster than standard whole-image/whole-transcript techniques
- Resolved (merged) segmentation shows no duplicated or split cells at patch boundaries — polygons merged over all patches remove potential artefacts
- Segmentation runs complete on datasets with millions of cells without exhausting memory on a high-performance computational cluster

## Limitations

- Overlapping patches introduce boundary artefacts that must be resolved by merging polygons over patches; conflicts at overlaps are an inherent cost of tiling
- Most existing segmentation methods are not designed to handle large images with millions of cells, demanding high-performance computational clusters with substantial memory resources — patching mitigates but assumes a SpatialData-compatible input pipeline
- Support is limited to specific backends (Cellpose for image-based, Baysor for transcript-based); future methods can be added once published and validated

## Evidence

- [intro] Patching motivation: "Most existing methods18–20 are not designed to handle large images with millions of cells. Their usage typically demands high-performance computational clusters with substantial memory resources"
- [intro] Patch generation: "we generate overlapping patches of images and/or transcripts."
- [intro] Per-patch segmentation backends: "Segmentation can then be performed for each individual patch, and we currently support Cellpose18 (image-based segmentation) and Baysor19 (transcripts-based segmentation)."
- [intro] Boundary resolution: "the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts."
- [readme] Patchify command with parameters: [evidence span withheld — anchor status: not_found]
- [readme] Expected speedup: [evidence span withheld — anchor status: not_found]
