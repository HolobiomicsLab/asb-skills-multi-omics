---
name: segmentation-ratio-computation
description: Use when evaluating whether segmentation of large spatial omics images
  (e.g. Xenium, MERSCOPE) hits memory/time limits, or when quantifying how much Sopa
  patching reduces RAM and runtime versus whole-image/whole-transcript segmentation.
when_to_use_negative:
- Input images or transcript sets are small enough to segment without patching — the
  benchmark's purpose (RAM/time scalability on millions of cells) does not apply
- Data are already segmented (cell boundaries provided) — Sopa's patch/resolve segmentation
  pipeline assumes raw images and/or transcript coordinates
- The technology provides neither image channels nor transcript coordinates usable
  by Cellpose or Baysor
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: Sopa
  role: Provides the patchify/resolve CLI and segmentation orchestration used to run
    patched vs unpatched Cellpose and Baysor benchmarks
  repo: https://github.com/gustaveroussy/sopa
- name: Cellpose
  role: Image-based segmentation model benchmarked on whole image (baseline) and per
    1500-px overlapping patch (patched, diameter 35, min_area 2000)
- name: Baysor
  role: Transcript-based segmentation model benchmarked on the whole transcript set
    (baseline) and per 200-micron transcript patch (patched, scale=6 microns, min_molecules_per_cell=10)
- name: SpatialData
  role: Universal data framework holding the loaded Xenium/MERSCOPE crops and toy
    dataset on which all segmentation runs are performed
  repo: https://github.com/scverse/spatialdata
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:36d45fabf5f7e85b84e690a85db5336c792da616fc60e738ea3cef564bad7c62
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
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# segmentation-ratio-computation

## Summary

Benchmarks Cellpose (image-based) and Baysor (transcripts-based) segmentation with and without Sopa's overlapping-patch strategy on SpatialData objects, computing peak-RAM and wall-clock-time ratios between unpatched baselines and patched runs to quantify the claimed 10-100x memory savings and up to 100x speedup.

## When to use

Use when evaluating whether segmentation of large spatial omics images (e.g. Xenium, MERSCOPE) hits memory/time limits, or when quantifying how much Sopa patching reduces RAM and runtime versus whole-image/whole-transcript segmentation.

## When NOT to use

- Input images or transcript sets are small enough to segment without patching — the benchmark's purpose (RAM/time scalability on millions of cells) does not apply
- Data are already segmented (cell boundaries provided) — Sopa's patch/resolve segmentation pipeline assumes raw images and/or transcript coordinates
- The technology provides neither image channels nor transcript coordinates usable by Cellpose or Baysor

## Inputs

- MERSCOPE or Xenium data crop loaded as a SpatialData object (via sopa.io.merscope / sopa.io.xenium), or a synthetic dataset from sopa.io.toy_dataset
- Whole-image or whole-transcript set for unpatched baseline segmentation
- Baysor config.toml (scale=6 microns, min_molecules_per_cell=10)
- Patch geometry parameters (patch-width-pixel 1500, patch-overlap-pixel 50; patch-width-microns 200)

## Outputs

- benchmark_results.csv containing memory-ratio and time-ratio entries for Cellpose and Baysor on each dataset
- Patched segmentation masks as cell polygons merged across patches (sopa resolve cellpose/baysor)
- Peak RAM (bytes) and wall-clock time measurements per run

## How to apply

Install sopa from the cloned repository in editable/dev mode with 'uv sync --all-extras --dev'. Generate a synthetic dataset via sopa.io.toy_dataset or load MERSCOPE/Xenium crops into SpatialData objects using sopa.io readers (e.g. sopa.io.merscope, sopa.io.xenium). Run unpatched baselines: Cellpose over the whole image and Baysor over the whole transcript set, recording peak RAM and wall-clock time. Then run the patch strategy: 'sopa patchify image --patch-width-pixel 1500 --patch-overlap-pixel 50' followed by Cellpose (diameter 35, min_area 2000) and 'sopa patchify transcripts --patch-width-microns 200' with Baysor (scale=6 microns, min_molecules_per_cell=10 via config.toml), then 'sopa resolve cellpose' / 'sopa resolve baysor' to merge cell polygons across patch boundaries and remove artefacts. Compute memory-ratio and time-ratio (unpatched/patched) per method and dataset, and write all measured ratios to benchmark_results.csv.

## Related tools

- **Sopa** (Provides the patchify/resolve CLI and segmentation orchestration used to run patched vs unpatched Cellpose and Baysor benchmarks) — https://github.com/gustaveroussy/sopa
- **Cellpose** (Image-based segmentation model benchmarked on whole image (baseline) and per 1500-px overlapping patch (patched, diameter 35, min_area 2000))
- **Baysor** (Transcript-based segmentation model benchmarked on the whole transcript set (baseline) and per 200-micron transcript patch (patched, scale=6 microns, min_molecules_per_cell=10))
- **SpatialData** (Universal data framework holding the loaded Xenium/MERSCOPE crops and toy dataset on which all segmentation runs are performed) — https://github.com/scverse/spatialdata

## Examples

```
sopa patchify image sdata --patch-width-pixel 1500 --patch-overlap-pixel 50 && sopa resolve cellpose sdata
```

## Evaluation signals

- benchmark_results.csv exists and contains memory-ratio and time-ratio entries for both Cellpose and Baysor on each dataset
- Patched runs show substantially lower peak RAM than the unpatched Cellpose/Baysor baselines, consistent with 10-100x memory savings
- Patched runtimes are up to ~100x faster than unpatched runs, matching the reported speedup
- Cell polygons from overlapping patches are successfully merged by 'sopa resolve cellpose/baysor' without boundary artefacts
- Patch geometry matches the specified parameters (1500-px width, 50-px overlap for images; 200-micron width for transcripts)

## Limitations

- Ratios depend on dataset size and hardware; the reported 10-100x memory and up to 100x time savings are order-of-magnitude claims rather than fixed constants
- No software version numbers for Sopa or its dependencies are reported in the article's discussion, so reproducibility of exact ratios requires pinning versions in the dev environment
- Merging masks over patches requires resolving potential artefacts at patch boundaries; heavy overlap increases the resolve step's cost

## Evidence

- [intro] Segmentation can then be performed for each individual patch, and we currently support Cellpose18 (image-based segmentation) and Baysor19 (transcripts-based segmentation).: "Segmentation can then be performed for each individual patch, and we currently support Cellpose18 (image-based segmentation) and Baysor19 (transcripts-based segmentation)."
- [intro] Most existing methods are not designed to handle large images with millions of cells, demanding high-performance computational clusters with substantial memory.: "Most existing methods18–20 are not designed to handle large images with millions of cells. Their usage typically demands high-performance computational clusters with substantial memory resources"
- [abstract] We introduce Sopa, a technology-invariant, memory-efficient pipeline with a unified visualizer for all image-based spatial omics.: "We introduce Sopa ( https://github.com/gustaveroussy/sopa ), a technology-invariant, memory-efficient pipeline with a unified visualizer for all image-based spatial omics."
- [intro] The cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts.: "the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts."
