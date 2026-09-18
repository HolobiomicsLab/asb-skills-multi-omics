---
name: segmentation-labels-layer-generation
description: Use when a raw spatial transcriptomics image (e.g. the RESOLVE mouse
  liver example) must be converted into a per-cell labels layer before transcript
  allocation, and a reproducible Cellpose-based segmentation config is needed.
when_to_use_negative:
- The SpatialData object already contains a validated cell segmentation labels layer
  and only downstream steps (transcript allocation, tb.bin_counts) are needed
- Segmentation must be driven by point/nucleus detections rather than image-based
  inference — use im.segment_points instead of image segmentation via im.segment
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: SPArrOW
  role: Pipeline library providing the im.segment stage (with im.segment_points, cellpose_callable,
    baysor_callable backends) that produces the cell labels layer
  repo: https://github.com/saeyslab/napari-sparrow
- name: uv
  role: Creates the virtual environment and installs dependencies (uv sync) including
    the dev/tiling extras needed for segmentation backends
  repo: https://docs.astral.sh/uv/
- name: Hydra
  role: Saves the configuration .yaml file for the segmentation step to the output
    directory and runs experiments from the CLI
  repo: https://hydra.cc
- name: Napari
  role: Optional plugin used to tune the parameters of the SPArrOW segmentation step
    on small crops of the image
  repo: https://github.com/saeyslab/napari-sparrow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.07.04.601829
    title: 'SPArrOW: a flexible, interactive and scalable pipeline for spatial transcriptomics
      analysis'
  source_content_hash: sha256:397d5ca63fb54b236be63088831f46c0cca06d570241b3876c9139ffbeb9599b
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.38461538461538464
n_claims: 13
---

# segmentation-labels-layer-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates a cell segmentation labels layer from a spatial transcriptomics image using SPArrOW's im.segment stage (BaSiC background estimation followed by Cellpose), storing the resulting mask back into the SpatialData object. Each pipeline step also writes a Hydra .yaml configuration file to the chosen output directory.

## When to use

Use when a raw spatial transcriptomics image (e.g. the RESOLVE mouse liver example) must be converted into a per-cell labels layer before transcript allocation, and a reproducible Cellpose-based segmentation config is needed.

## When NOT to use

- The SpatialData object already contains a validated cell segmentation labels layer and only downstream steps (transcript allocation, tb.bin_counts) are needed
- Segmentation must be driven by point/nucleus detections rather than image-based inference — use im.segment_points instead of image segmentation via im.segment

## Inputs

- SpatialData object with an image layer (e.g. sp.datasets.resolve_example() RESOLVE mouse liver image)
- SPArrOW experiment configuration (configs/experiment/resolve_liver.yaml)
- Installed SPArrOW environment with dev/tiling extras (Cellpose/PyTorch backends)

## Outputs

- Cell segmentation labels layer stored in the SpatialData object as the segmentation mask
- Hydra .yaml configuration file saved in the user-chosen output directory

## How to apply

Install the SPArrOW environment with uv (uv sync), ensuring the dev/tiling extras are included so segmentation backends such as Cellpose/PyTorch are available rather than skipped. Load the spatial object with sp.datasets.resolve_example() from the sparrow.datasets module, or use the RESOLVE mouse liver experiment preconfigured in configs/experiment/resolve_liver.yaml. Run im.segment from the sparrow.im subpackage on the loaded image layer (im.segment_points is the point-based alternative; cellpose_callable/baysor_callable are the backends) to generate a cell labels layer, applying BaSiC background estimation as part of the segmentation stage. Store the resulting labels layer back into the SpatialData object as the segmentation mask. After the step, record the Hydra .yaml configuration file that the pipeline saves to the chosen output directory so the run is reproducible.

## Related tools

- **SPArrOW** (Pipeline library providing the im.segment stage (with im.segment_points, cellpose_callable, baysor_callable backends) that produces the cell labels layer) — https://github.com/saeyslab/napari-sparrow
- **uv** (Creates the virtual environment and installs dependencies (uv sync) including the dev/tiling extras needed for segmentation backends) — https://docs.astral.sh/uv/
- **Hydra** (Saves the configuration .yaml file for the segmentation step to the output directory and runs experiments from the CLI) — https://hydra.cc
- **Napari** (Optional plugin used to tune the parameters of the SPArrOW segmentation step on small crops of the image) — https://github.com/saeyslab/napari-sparrow

## Examples

```
uv sync --extra dev --extra tiling; uv run python -c "import sparrow as sp; sdata = sp.datasets.resolve_example(); sdata = sp.im.segment(sdata)"
```

## Evaluation signals

- The segmentation labels layer is produced for the example image and present in the SpatialData object as the segmentation mask
- A corresponding .yaml configuration file is present in the chosen output directory after the segmentation step
- The im.segment stage ran with the Cellpose/PyTorch backend available (dev/tiling extras installed), not skipped
- Parameter tuning via the Napari plugin on small crops of the image confirms plausible cell boundaries before full-image segmentation

## Limitations

- The package is still under very active development, so APIs and behavior may change
- If the dev/tiling extras are not installed during uv sync, segmentation backends such as Cellpose/PyTorch are skipped
- Experiments run locally by default; on a SLURM cluster the Hydra launcher must be changed to submitit_local overrides
- The article preprint has no discussion-section content or changelog available for further caveats

## Evidence
