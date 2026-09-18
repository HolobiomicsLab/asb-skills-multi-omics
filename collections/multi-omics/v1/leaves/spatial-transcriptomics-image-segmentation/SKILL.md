---
name: spatial-transcriptomics-image-segmentation
description: Use when raw spatial transcriptomics imaging data (e.g. RESOLVE, MERSCOPE,
  Xenium, Visium HD image layers in a SpatialData object) must be converted into per-cell
  segmentation masks before transcript allocation or downstream clustering.
when_to_use_negative:
- The input is already a segmented labels layer or a cell x gene feature table — proceed
  to transcript allocation (tb.allocate/tb.bin_counts) or preprocessing instead
- Transcripts are unallocated points without a segmentation mask but the goal is only
  point-based binning rather than cell-shape segmentation — consider im.segment_points
  for point-based segmentation instead of image-based Cellpose
- Parameter values are unknown and need interactive tuning — use the napari-sparrow
  plugin to tune parameters on small image crops before running the CLI pipeline
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SPArrOW
  role: Pipeline library whose sparrow.im subpackage (im.segment, im.segment_points)
    performs the cell segmentation step
  repo: https://github.com/saeyslab/napari-sparrow
- name: uv
  role: Creates the virtual environment and installs segmentation dependencies (Cellpose/PyTorch
    via dev/tiling extras)
- name: Hydra
  role: Runs the segmentation experiment from the CLI and saves the .yaml configuration
    for each step
  repo: https://hydra.cc
- name: Napari
  role: Interactive plugin used to tune segmentation parameters on small crops of
    the image before full runs
  repo: https://github.com/napari/napari
- name: Python
  role: Runtime; SPArrOW requires Python 3.11 or newer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.07.04.601829
    title: 'SPArrOW: a flexible, interactive and scalable pipeline for spatial transcriptomics
      analysis'
  source_content_hash: sha256:fa5c12775a6177d7c9217f251c46d3be6b5368cb083d83c4fb99c244210d8a5a
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# spatial-transcriptomics-image-segmentation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Segments cells in spatial transcriptomics microscopy images (e.g. the RESOLVE mouse liver example) using the SPArrOW im.segment stage, which pairs BaSiC tiling correction/background estimation with Cellpose nuclei-based segmentation to produce a cell labels layer inside a SpatialData object.

## When to use

Use when raw spatial transcriptomics imaging data (e.g. RESOLVE, MERSCOPE, Xenium, Visium HD image layers in a SpatialData object) must be converted into per-cell segmentation masks before transcript allocation or downstream clustering.

## When NOT to use

- The input is already a segmented labels layer or a cell x gene feature table — proceed to transcript allocation (tb.allocate/tb.bin_counts) or preprocessing instead
- Transcripts are unallocated points without a segmentation mask but the goal is only point-based binning rather than cell-shape segmentation — consider im.segment_points for point-based segmentation instead of image-based Cellpose
- Parameter values are unknown and need interactive tuning — use the napari-sparrow plugin to tune parameters on small image crops before running the CLI pipeline

## Inputs

- SpatialData object with an image layer (e.g. RESOLVE mouse liver example from sp.datasets.resolve_example())
- Hydra experiment configuration .yaml (e.g. configs/experiment/resolve_liver.yaml)
- Docker-less Python 3.11+ environment created with uv including dev/tiling extras

## Outputs

- Cell labels layer (segmentation mask) stored in the SpatialData object
- Hydra .yaml configuration file written to the chosen output directory
- Labels layer suitable for downstream transcript allocation (tb.allocate) and binning

## How to apply

Install the SPArrOW environment with uv (`uv sync` including dev/tiling extras so Cellpose/PyTorch backends are not skipped), requiring Python 3.11 or newer. Load a bundled example via `sparrow.datasets` (e.g. `sp.datasets.resolve_example()` for the RESOLVE mouse liver experiment, preconfigured in `configs/experiment/resolve_liver.yaml`). Apply image preprocessing such as tiling correction (BaSiC) and contrast enhancement to the image layer, then run `im.segment` from the `sparrow.im` subpackage — with `im.segment_points` for point-based cases and the `cellpose_callable` backend — to generate a cell labels layer. Store the resulting labels layer back into the SpatialData object as the segmentation mask, and note that Hydra saves a corresponding .yaml configuration file to the chosen output directory after every step, which should be retained for reproducibility.

## Related tools

- **SPArrOW** (Pipeline library whose sparrow.im subpackage (im.segment, im.segment_points) performs the cell segmentation step) — https://github.com/saeyslab/napari-sparrow
- **uv** (Creates the virtual environment and installs segmentation dependencies (Cellpose/PyTorch via dev/tiling extras))
- **Hydra** (Runs the segmentation experiment from the CLI and saves the .yaml configuration for each step) — https://hydra.cc
- **Napari** (Interactive plugin used to tune segmentation parameters on small crops of the image before full runs) — https://github.com/napari/napari
- **Python** (Runtime; SPArrOW requires Python 3.11 or newer)

## Examples

```
uv sync --extra dev --extra tiling; uv run sparrow im.segment datasets=resolve_example
```

## Evaluation signals

- A cell labels layer is produced and stored in the SpatialData object after im.segment completes
- A .yaml configuration file corresponding to the segmentation step is present in the chosen output directory
- The environment was synced with dev/tiling extras so the Cellpose/PyTorch backend is available rather than skipped
- The labels layer can be loaded and visualized/inspected (e.g. in Napari) over the tiling-corrected image of the example dataset

## Limitations

- The package is still under very active development, so APIs and defaults may change
- Requires Python 3.11 or newer and uv for environment management
- The experiment is designed for SLURM clusters via Submitit as well as local runs; resource assumptions may differ between the two

## Evidence
