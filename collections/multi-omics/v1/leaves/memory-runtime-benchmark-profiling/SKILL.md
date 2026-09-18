---
name: memory-runtime-benchmark-profiling
description: Use when evaluating whether segmentation of large image-based spatial
  omics data (e.g. Xenium, MERSCOPE) is memory/time-limited, or when validating that
  Sopa's patch-based strategy reduces peak RAM and runtime versus whole-image baselines.
when_to_use_negative:
- The dataset is small enough that Cellpose or Baysor already runs within available
  RAM without patching — the benchmark's memory savings will not be meaningful.
- The goal is segmentation quality assessment rather than scalability profiling —
  patching artefacts and ratio metrics do not measure segmentation accuracy.
- Input is not image-based spatial omics (e.g. already-segmented cell tables or non-spatial
  single-cell data); Sopa's patchify/resolve workflow does not apply.
edam_operation: http://edamontology.org/operation_3875
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: Sopa
  role: Provides the patchify (image/transcript), segment, and resolve commands used
    to run patched vs unpatched segmentation and merge patch boundaries
  repo: https://github.com/gustaveroussy/sopa
- name: Cellpose
  role: Image-based segmentation benchmarked whole-image (unpatched) and per overlapping
    image patch (diameter 35, min_area 2000)
- name: Baysor
  role: Transcript-based segmentation benchmarked on the full transcript set vs per
    transcript patch (scale=6 microns, min_molecules_per_cell=10)
  repo: https://github.com/kharchenkolab/Baysor
- name: SpatialData
  role: Unified data framework holding the input images/transcripts loaded via sopa.io
    readers
  repo: https://github.com/scverse/spatialdata
- name: uv
  role: Used to install sopa in editable/dev mode with 'uv sync --all-extras --dev'
    before benchmarking
  repo: https://github.com/astral-sh/uv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:4feb42d8bee1a8a2092274b32fd0756c35fd90d696b8aa1612dd627c24114c1c
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
  - uv
  edam_operation: http://edamontology.org/operation_3875
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# memory-runtime-benchmark-profiling

## Summary

Benchmark the peak RAM and wall-clock time of segmentation algorithms (Cellpose on images, Baysor on transcripts) run unpatched versus with Sopa's overlapping-patch strategy, quantifying the memory and time reduction ratios on large spatial omics data.

## When to use

Use when evaluating whether segmentation of large image-based spatial omics data (e.g. Xenium, MERSCOPE) is memory/time-limited, or when validating that Sopa's patch-based strategy reduces peak RAM and runtime versus whole-image baselines.

## When NOT to use

- The dataset is small enough that Cellpose or Baysor already runs within available RAM without patching — the benchmark's memory savings will not be meaningful.
- The goal is segmentation quality assessment rather than scalability profiling — patching artefacts and ratio metrics do not measure segmentation accuracy.
- Input is not image-based spatial omics (e.g. already-segmented cell tables or non-spatial single-cell data); Sopa's patchify/resolve workflow does not apply.

## Inputs

- Image-based spatial omics data as SpatialData objects (e.g. MERSCOPE or Xenium crops loaded via sopa.io.merscope / sopa.io.xenium)
- Synthetic test data generated with sopa.io.toy_dataset
- Baysor config.toml with scale=6 microns and min_molecules_per_cell=10
- Unpatched and patched segmentation runs with recorded peak RAM and wall-clock time

## Outputs

- benchmark_results.csv containing memory-ratio and time-ratio entries for Cellpose and Baysor per dataset
- Patch-resolved segmentation masks/merged cell polygons after 'sopa resolve'
- Peak RAM (GB) and wall-clock time measurements for each unpatched/patched run

## How to apply

Install sopa from the cloned repository in editable/dev mode ('uv sync --all-extras --dev') and prepare inputs as SpatialData objects via sopa.io readers (sopa.io.merscope, sopa.io.xenium) or a synthetic sopa.io.toy_dataset. Run unpatched baselines first: Cellpose on the whole image and Baysor on the whole transcript set, recording peak RAM and wall-clock time for each. Then repeat with the overlapping-patch strategy: 'sopa patchify image ... --patch-width-pixel 1500 --patch-overlap-pixel 50' for Cellpose (diameter 35, min_area 2000) and 'sopa patchify transcripts ... --patch-width-microns 200' for Baysor (config.toml with scale=6 microns, min_molecules_per_cell=10), followed by 'sopa resolve cellpose' or 'sopa resolve baysor' to merge patch boundaries and remove artefacts. Compute the unpatched-vs-patched memory ratio and time ratio per method and dataset, and write the measured values to benchmark_results.csv. Rationale: Sopa applies segmentation models to smaller overlapping patches, requiring 10-100x less memory and being up to 100x faster than running Cellpose/Baysor on the full image or full transcript set.

## Related tools

- **Sopa** (Provides the patchify (image/transcript), segment, and resolve commands used to run patched vs unpatched segmentation and merge patch boundaries) — https://github.com/gustaveroussy/sopa
- **Cellpose** (Image-based segmentation benchmarked whole-image (unpatched) and per overlapping image patch (diameter 35, min_area 2000))
- **Baysor** (Transcript-based segmentation benchmarked on the full transcript set vs per transcript patch (scale=6 microns, min_molecules_per_cell=10)) — https://github.com/kharchenkolab/Baysor
- **SpatialData** (Unified data framework holding the input images/transcripts loaded via sopa.io readers) — https://github.com/scverse/spatialdata
- **uv** (Used to install sopa in editable/dev mode with 'uv sync --all-extras --dev' before benchmarking) — https://github.com/astral-sh/uv

## Examples

```
sopa patchify image ... --patch-width-pixel 1500 --patch-overlap-pixel 50; sopa patchify transcripts ... --patch-width-microns 200
```

## Evaluation signals

- benchmark_results.csv exists and contains memory-ratio and time-ratio entries for both Cellpose and Baysor on each tested dataset.
- Patched runs show substantially lower peak RAM than unpatched baselines, consistent with the reported 10-100x memory savings.
- Patched runtime is faster than the unpatched run, consistent with up to 100x speedup reported for Sopa's patch strategy.
- After 'sopa resolve', merged cell boundaries show no artefacts at patch borders (segmentation masks are converted to polygons and merged over patches).
- Peak-RAM and wall-clock measurements are recorded for both the baseline (no patching) and patched runs of each method, not just one condition.

## Limitations

- Most existing spatial omics methods are not designed to handle large images with millions of cells, so unpatched baselines may require high-performance computational clusters with substantial memory resources.
- Reported savings depend on the chosen patch parameters (e.g. --patch-width-pixel 1500, --patch-overlap-pixel 50, --patch-width-microns 200); other settings will shift the measured ratios.
- Integration of future segmentation/annotation methods into the benchmark is contingent on their publication and validation.

## Evidence

- [intro] Sopa applies segmentation models to smaller overlapping patches, drastically decreasing RAM usage and time, requiring 10-100x less memory and being up to 100x faster than standard techniques.: [evidence span withheld — anchor status: not_found]
- [intro] Patch generation and per-patch segmentation workflow: "we generate overlapping patches of images and/or transcripts."
- [intro] Merging masks over patches to remove artefacts: "the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts."
- [intro] Scalability motivation: existing methods cannot handle large images without clusters: "Their usage typically demands high-performance computational clusters with substantial memory resources"
- [abstract] Sopa is a memory-efficient pipeline across image-based spatial omics: "We introduce Sopa ( https://github.com/gustaveroussy/sopa ), a technology-invariant, memory-efficient pipeline"
- [methods] Toy dataset used for tutorial benchmarks: "For this tutorial, we use a generated dataset. You can expect a total runtime of a few minutes."
