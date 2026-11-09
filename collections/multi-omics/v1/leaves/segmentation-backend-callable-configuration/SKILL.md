---
name: segmentation-backend-callable-configuration
description: Use when a spatial transcriptomics pipeline needs cell/nucleus segmentation
  for images (e.g. the RESOLVE mouse liver example) and the segmentation backend (Cellpose
  or Baysor) and its parameters must be configured and validated within SPArrOW.
when_to_use_negative:
- The input is already a cell-by-gene count matrix or feature table with existing
  segmentation labels (skip to transcript allocation / tb.allocate instead)
- The analysis targets transcripts without an accompanying image (use point-based
  segmentation such as im.segment_points or Baysor rather than image-based Cellpose
  segmentation)
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SPArrOW
  role: Pipeline providing the im.segment / im.segment_points stage with cellpose_callable
    and baysor_callable backends for single-cell spatial transcriptomics data analysis
  repo: https://github.com/saeyslab/napari-sparrow
- name: uv
  role: Creates the virtual environment and installs dependencies, including segmentation-backend
    extras
  repo: https://docs.astral.sh/uv/
- name: Hydra
  role: Manages and saves the per-step configuration .yaml file produced after segmentation;
    runs experiments locally or on SLURM
  repo: https://hydra.cc
- name: Napari
  role: Plugin used to tune segmentation parameters on small crops of the image
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.07.04.601829
    title: 'SPArrOW: a flexible, interactive and scalable pipeline for spatial transcriptomics
      analysis'
  source_content_hash: sha256:f46fa7458602cfa4f7059c9538a1c9151c637c1f221aeb4670a47f20e6585f37
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/saeyslab/napari-SPArrOW
  repo_url: https://github.com/saeyslab/napari-sparrow
  tools:
  - Hydra
  - Napari
  - Python
  - SPArrOW
  - uv
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# segmentation-backend-callable-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configuring and running the SPArrOW segmentation stage (im.segment with cellpose_callable/baysor_callable backends) on spatial-transcriptomics images, including background estimation via BaSiC, to produce a cell labels layer and a saved Hydra .yaml configuration. It exists because SPArrOW's segmentation backends are selected and parameterized through pluggable callables whose outputs must be re-registered as a SpatialData segmentation mask.

## When to use

Use when a spatial transcriptomics pipeline needs cell/nucleus segmentation for images (e.g. the RESOLVE mouse liver example) and the segmentation backend (Cellpose or Baysor) and its parameters must be configured and validated within SPArrOW.

## When NOT to use

- The input is already a cell-by-gene count matrix or feature table with existing segmentation labels (skip to transcript allocation / tb.allocate instead)
- The analysis targets transcripts without an accompanying image (use point-based segmentation such as im.segment_points or Baysor rather than image-based Cellpose segmentation)

## Inputs

- SpatialData object with an image layer (e.g. the RESOLVE mouse liver example from sp.datasets.resolve_example())
- Hydra experiment configuration (e.g. configs/experiment/resolve_liver.yaml)
- Installed SPArrOW environment with Cellpose/PyTorch segmentation backend (uv sync with dev/tiling extras)

## Outputs

- Cell segmentation labels layer stored back into the SpatialData object as the segmentation mask
- Hydra configuration .yaml file saved in the user-chosen output directory after the segmentation step

## How to apply

Set up the SPArrOW project environment with uv (uv sync), making sure the dev/tiling extras are installed so segmentation backends such as Cellpose/PyTorch are available rather than skipped. Load a spatial object via sparrow.datasets (e.g. sp.datasets.resolve_example() for the RESOLVE mouse liver image, which is preconfigured in configs/experiment/resolve_liver.yaml). Call the segmentation module of the sparrow.im subpackage (im.segment, or im.segment_points for point-based segmentation) choosing the cellpose_callable or baysor_callable backend, and optionally run im.tiling_correction/enhance_contrast preprocessing first. Store the returned labels layer back into the SpatialData object as the segmentation mask, and record the Hydra .yaml file that the pipeline saves to the chosen output directory after the step. Parameters can be tuned on small image crops via the napari-sparrow plugin before running at scale (locally or on a SLURM cluster via Submitit).

## Related tools

- **SPArrOW** (Pipeline providing the im.segment / im.segment_points stage with cellpose_callable and baysor_callable backends for single-cell spatial transcriptomics data analysis) — https://github.com/saeyslab/napari-sparrow
- **uv** (Creates the virtual environment and installs dependencies, including segmentation-backend extras) — https://docs.astral.sh/uv/
- **Hydra** (Manages and saves the per-step configuration .yaml file produced after segmentation; runs experiments locally or on SLURM) — https://hydra.cc
- **Napari** (Plugin used to tune segmentation parameters on small crops of the image)

## Examples

```
uv sync --extra dev --extra tiling && uv run python -c "import sparrow as sp; sdata = sp.datasets.resolve_example(); sp.im.segment(sdata, img_layer='resolve_image')"
```

## Evaluation signals

- A segmentation labels layer is produced for the example image and is stored back in the SpatialData object as the segmentation mask
- A corresponding Hydra .yaml configuration file is present in the chosen output directory after the segmentation step
- The uv environment resolves with the dev/tiling extras so Cellpose/PyTorch backends are imported rather than skipped
- The run uses the preconfigured RESOLVE mouse liver experiment config (configs/experiment/resolve_liver.yaml) without schema/config errors
- Tuning on small image crops via the napari-sparrow plugin yields the same parameters as those written to the saved .yaml file

## Limitations

- The package is still under very active development, so APIs such as im.segment and the callable backends may change without notice
- Segmentation backends such as Cellpose/PyTorch may be skipped if the dev/tiling extras are not installed during uv sync
- Configurations and outputs depend on Hydra; experiments must be rerun with correct launcher settings if moved to a SLURM cluster (submitit_local override)

## Evidence

- [methods] SPArrOW is a library for single-cell Spatial Transcriptomics data analysis.: "SPArrOW is a library for single-cell Spatial Transcriptomics data analysis."
- [readme] Note: This package is still under very active development.: "Note: This package is still under very active development."
- [methods] After every step, a corresponding configuration _.yaml_ file will be saved in the output directory chosen by the user.: "After every step, a corresponding configuration _.yaml_ file will be saved in the output directory chosen by the user."
- [methods] The RESOLVE mouse liver experiment is preconfigured in `configs/experiment/resolve_liver.yaml`: "The RESOLVE mouse liver experiment is preconfigured in `configs/experiment/resolve_liver.yaml`"
- [methods] im.segment, im.segment_points: "im.segment
    im.segment_points"
