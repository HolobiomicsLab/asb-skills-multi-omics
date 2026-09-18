---
name: hydra-configuration-file-generation
description: Use when running SPArrOW pipeline steps (e.g. im.segment on a resolve_example
  spatial object) and you need to locate and archive the Hydra-generated .yaml config
  that encodes that step's parameters.
when_to_use_negative:
- You are using SPArrOW only interactively as a Napari plugin to tune parameters on
  small image crops rather than running reproducible pipeline steps
- The task requires modifying pipeline parameters rather than recording the auto-saved
  per-step configuration
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Hydra
  role: Framework through which SPArrOW experiments are run from the CLI, generating
    and saving the per-step .yaml configuration files
  repo: https://hydra.cc
- name: SPArrOW
  role: Pipeline library whose steps (e.g. im.segment) write the configuration .yaml
    to the chosen output directory
  repo: https://github.com/saeyslab/napari-sparrow
- name: SLURM
  role: Cluster launcher option for Hydra-run experiments, changing the hydra/launcher
    configuration
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.07.04.601829
    title: 'SPArrOW: a flexible, interactive and scalable pipeline for spatial transcriptomics
      analysis'
  source_content_hash: sha256:a968b385d65d5831bc091371ef8d92d4757e2dd7f94b8554c1d42f4edeb5e5c6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/saeyslab/napari-SPArrOW
  tools:
  - Hydra
  - Python
  - SLURM
  - SPArrOW
  - uv
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# hydra-configuration-file-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Captures and records the Hydra .yaml configuration file that the SPArrOW pipeline saves to the user-chosen output directory after every pipeline step (e.g. segmentation), making each experiment reproducible and parameter traceable.

## When to use

Use when running SPArrOW pipeline steps (e.g. im.segment on a resolve_example spatial object) and you need to locate and archive the Hydra-generated .yaml config that encodes that step's parameters.

## When NOT to use

- You are using SPArrOW only interactively as a Napari plugin to tune parameters on small image crops rather than running reproducible pipeline steps
- The task requires modifying pipeline parameters rather than recording the auto-saved per-step configuration

## Inputs

- SPArrOW pipeline step execution (e.g. sparrow.im.segment on a SpatialData object loaded via sp.datasets.resolve_example())
- User-chosen output directory
- Hydra experiment config (e.g. configs/experiment/resolve_liver.yaml)

## Outputs

- Hydra configuration .yaml file saved in the output directory after the executed step

## How to apply

SPArrOW runs experiments from the CLI using Hydra, locally or on a SLURM cluster. After each pipeline step — such as im.tiling_correction, im.segment, or tb.allocate — the pipeline writes a corresponding configuration .yaml file into the output directory chosen by the user. Locate that .yaml in the output directory and treat it as the parameter record for the step; the RESOLVE mouse liver experiment is preconfigured via configs/experiment/resolve_liver.yaml, which can be used as the experiment config basis. Validation is simple: the config file must exist in the output directory and correspond to the executed step.

## Related tools

- **Hydra** (Framework through which SPArrOW experiments are run from the CLI, generating and saving the per-step .yaml configuration files) — https://hydra.cc
- **SPArrOW** (Pipeline library whose steps (e.g. im.segment) write the configuration .yaml to the chosen output directory) — https://github.com/saeyslab/napari-sparrow
- **SLURM** (Cluster launcher option for Hydra-run experiments, changing the hydra/launcher configuration)

## Examples

```
uv run sparrow im.segment +experiment=resolve_liver  # then check output dir for the saved .yaml
```

## Evaluation signals

- A .yaml configuration file exists in the chosen output directory after the pipeline step completes
- The .yaml file corresponds to the executed step (e.g. segmentation) rather than another stage
- Config content matches the experiment config used (e.g. derived from configs/experiment/resolve_liver.yaml for the RESOLVE mouse liver experiment)
- The config persists alongside the step's data outputs (e.g. the segmentation labels layer stored in the SpatialData object)

## Limitations

- SPArrOW is still under very active development, so configuration file behavior may change between versions
- Documentation does not detail the internal schema of the generated .yaml files

## Evidence

- [methods] Hydra config generation per step: "After every step, a corresponding configuration _.yaml_ file will be saved in the output directory chosen by the user."
- [methods] Hydra runs: "Run experiments from the CLI using [Hydra](https://hydra.cc). Experiments can be run locally, or on a SLURM cluster."
- [readme] Development status: "Note: This package is still under very active development."
