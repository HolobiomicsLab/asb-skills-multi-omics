---
name: spatialdata-example-dataset-loading
description: Use when you need a small, preconfigured spatial transcriptomics test
  object (e.g. the RESOLVE mouse liver image) to develop or validate SPArrOW stages
  like im.segment before running on your own data.
when_to_use_negative:
- You already have your own spatial transcriptomics data loaded into a SpatialData
  object — use the regular I/O path instead of an example loader.
- The target pipeline stage needs production-scale data rather than a small bundled
  demonstration image.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: SPArrOW
  role: Provides the sparrow.datasets module with example loaders (datasets.resolve_example
    etc.) returning SpatialData objects
  repo: https://github.com/saeyslab/napari-sparrow
- name: uv
  role: Creates the virtual environment and installs dependencies, including dev/tiling
    extras so segmentation backends are available
  repo: https://docs.astral.sh/uv/
- name: Python
  role: Runtime required (Python 3.11 or newer) in which the dataset loaders are called
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.07.04.601829
    title: 'SPArrOW: a flexible, interactive and scalable pipeline for spatial transcriptomics
      analysis'
  source_content_hash: sha256:23e784380a1280bac8971a06d7308e2ddc6ad7f1bb08c741669dbca7476c5bd1
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
  - Python
  - SPArrOW
  - uv
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.16666666666666666
n_claims: 6
---

# spatialdata-example-dataset-loading

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Load preconfigured example spatial transcriptomics datasets (RESOLVE mouse liver, MERSCOPE, Xenium, Visium HD, Vizgen) bundled with SPArrOW via the sparrow.datasets module, to exercise pipeline stages like segmentation on ready-to-run SpatialData objects.

## When to use

Use when you need a small, preconfigured spatial transcriptomics test object (e.g. the RESOLVE mouse liver image) to develop or validate SPArrOW stages like im.segment before running on your own data.

## When NOT to use

- You already have your own spatial transcriptomics data loaded into a SpatialData object — use the regular I/O path instead of an example loader.
- The target pipeline stage needs production-scale data rather than a small bundled demonstration image.

## Inputs

- Bundled example dataset name (e.g. resolve_example, merscope_example, xenium_example, visium_hd_example, vizgen_example)
- SPArrOW project environment installed with uv, including dev/tiling extras for segmentation backends (Cellpose/PyTorch)

## Outputs

- A loaded SpatialData object containing the example image layer (and associated layers)
- Downstream layers derived from the example (e.g. a cell labels/segmentation mask layer)

## How to apply

Import the datasets module from sparrow and call one of the registry example loaders — datasets.resolve_example, datasets.merscope_example, datasets.xenium_example, datasets.visium_hd_example, or datasets.vizgen_example — which download/open the bundled example into a SpatialData object. For the RESOLVE mouse liver experiment, the pipeline is preconfigured in configs/experiment/resolve_liver.yaml, so the loaded example pairs with that experiment configuration. Once loaded, the image layer can be passed to downstream SPArrOW subpackages such as im.segment (with im.segment_points and the cellpose_callable backend) to generate a cell labels layer, which is stored back into the SpatialData object as the segmentation mask. Ensure the environment includes the dev/tiling extras (installed via uv sync) so segmentation backends such as Cellpose/PyTorch are available rather than skipped.

## Related tools

- **SPArrOW** (Provides the sparrow.datasets module with example loaders (datasets.resolve_example etc.) returning SpatialData objects) — https://github.com/saeyslab/napari-sparrow
- **uv** (Creates the virtual environment and installs dependencies, including dev/tiling extras so segmentation backends are available) — https://docs.astral.sh/uv/
- **Python** (Runtime required (Python 3.11 or newer) in which the dataset loaders are called)

## Examples

```
from sparrow.datasets import resolve_example; sdata = resolve_example()
```

## Evaluation signals

- The example loader call returns a SpatialData object with a populated image layer (e.g. the RESOLVE mouse liver image).
- A downstream stage run on the loaded example (e.g. im.segment) produces a segmentation labels layer.
- The corresponding Hydra configuration .yaml file is saved in the chosen output directory after the pipeline step.
- The preconfigured experiment config configs/experiment/resolve_liver.yaml resolves without errors when paired with the loaded RESOLVE example.

## Limitations

- The SPArrOW package is still under very active development, so example dataset APIs and loader names may change.
- If the dev/tiling extras are omitted during uv sync, segmentation backends such as Cellpose/PyTorch may be skipped and downstream stages on the loaded example will fail.
- Examples are only useful for pipeline development/tuning; SPArrOW recommends tuning parameters on small crops of the image via the Napari plugin.

## Evidence
