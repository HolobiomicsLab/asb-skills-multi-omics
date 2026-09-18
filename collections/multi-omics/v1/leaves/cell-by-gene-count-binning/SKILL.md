---
name: cell-by-gene-count-binning
description: Use when you have segmented cell shapes (labels/polygons) plus raw transcript
  spot coordinates from a spatial transcriptomics run (e.g. the RESOLVE mouse liver
  dataset) and need a cell-by-gene count table.
when_to_use_negative:
- The input is already a cell-by-gene feature/count table — allocation and binning
  would be redundant.
- Segmentation has not yet been performed; there are no cell shapes for tb.allocate
  to assign transcripts to.
- The technology produces pixel- or spot-array intensities rather than single-transcript
  spot coordinates, so spot-to-polygon assignment does not apply.
tools:
- name: SPArrOW
  role: 'Provides the pipeline commands used in this skill: sh.vectorize, tb.allocate,
    tb.bin_counts, and tb.add_table_layer'
  repo: https://github.com/saeyslab/napari-sparrow
- name: napari-sparrow
  role: Napari plugin form of SPArrOW used to tune pipeline parameters (e.g. allocation)
    on small crops of the image
  repo: https://github.com/saeyslab/napari-sparrow
- name: Python
  role: Runtime for SPArrOW (Python 3.11 or newer), used to run the allocation and
    binning steps
- name: Hydra
  role: CLI/config framework used to run SPArrOW experiments such as the preconfigured
    resolve_liver experiment
  repo: https://hydra.cc
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.07.04.601829
    title: 'SPArrOW: a flexible, interactive and scalable pipeline for spatial transcriptomics
      analysis'
  source_content_hash: sha256:1378f55a46d708ecada8191fd63514253de6eef42ac62371006483e5c9676444
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
  - Python
  - SPArrOW
  - napari-sparrow
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.2857142857142857
n_claims: 7
---

# cell-by-gene-count-binning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

SPArrOW's transcript allocation stage: convert segmented cell labels into polygon shapes, assign detected transcript spots to overlapping cell polygons with tb.allocate, and bin the assignments into a cell-by-gene count matrix with tb.bin_counts, saved as a .h5ad AnnData table layer.

## When to use

Use when you have segmented cell shapes (labels/polygons) plus raw transcript spot coordinates from a spatial transcriptomics run (e.g. the RESOLVE mouse liver dataset) and need a cell-by-gene count table.

## When NOT to use

- The input is already a cell-by-gene feature/count table — allocation and binning would be redundant.
- Segmentation has not yet been performed; there are no cell shapes for tb.allocate to assign transcripts to.
- The technology produces pixel- or spot-array intensities rather than single-transcript spot coordinates, so spot-to-polygon assignment does not apply.

## Inputs

- Segmented cell shapes (labels layer or polygons) from the segmentation stage (e.g. im.segment output)
- Raw transcript spot coordinate file from the deposited dataset (e.g. RESOLVE mouse liver experiment transcripts)
- Hydra experiment configuration YAML (e.g. configs/experiment/resolve_liver.yaml)

## Outputs

- cell_by_gene_table.h5ad — AnnData object whose table layer holds a cell-by-gene count matrix (one row per segmented cell, one column per detected gene)

## How to apply

Load the segmented cell shapes from the segmentation output and the raw transcript spot coordinate file from the deposited dataset. Convert the labels layer into polygon shapes using SPArrOW's sh.vectorize, then assign each transcript spot to the overlapping cell polygon with tb.allocate — the first of the two consecutive steps SPArrOW uses for transcript allocation. Aggregate per-cell assignments into per-cell gene counts with tb.bin_counts, and attach the resulting matrix as a table layer with tb.add_table_layer. Write the AnnData object to disk as cell_by_gene_table.h5ad; SPArrOW experiments are driven by Hydra YAML configs (e.g. the preconfigured RESOLVE liver config at configs/experiment/resolve_liver.yaml) and can be tuned on small crops via the Napari plugin.

## Related tools

- **SPArrOW** (Provides the pipeline commands used in this skill: sh.vectorize, tb.allocate, tb.bin_counts, and tb.add_table_layer) — https://github.com/saeyslab/napari-sparrow
- **napari-sparrow** (Napari plugin form of SPArrOW used to tune pipeline parameters (e.g. allocation) on small crops of the image) — https://github.com/saeyslab/napari-sparrow
- **Python** (Runtime for SPArrOW (Python 3.11 or newer), used to run the allocation and binning steps)
- **Hydra** (CLI/config framework used to run SPArrOW experiments such as the preconfigured resolve_liver experiment) — https://hydra.cc

## Examples

```
uv run sparrow --config-name resolve_liver  # runs the SPArrOW pipeline (incl. tb.allocate + tb.bin_counts) preconfigured in configs/experiment/resolve_liver.yaml
```

## Evaluation signals

- The saved cell_by_gene_table.h5ad loads successfully as an AnnData object.
- The count matrix has exactly one row per segmented cell and one column per detected gene.
- The binned counts reproduce the counts implied by allocating the input transcript spots to the input cell polygons.
- The count matrix is attached as a table layer (added via tb.add_table_layer) within the AnnData object.
- The Hydra config for the run is written to the chosen output directory after execution, as SPArrOW saves a corresponding YAML per step.

## Limitations

- The napari-sparrow package is explicitly noted as 'still under very active development', so APIs and behavior may change.
- Allocation depends on the quality of the upstream segmentation; transcripts outside cell polygons are not captured in the table.
- The README provides no changelog; detailed per-step documentation must be sought in the readthedocs tutorials.

## Evidence

- [methods] The RESOLVE liver experiment is preconfigured: "The RESOLVE mouse liver experiment is preconfigured in `configs/experiment/resolve_liver.yaml`"
- [methods] SPArrOW is a library for single-cell spatial transcriptomics analysis: "SPArrOW is a library for single-cell Spatial Transcriptomics data analysis."
- [readme] Reproducibility scripts are published by the authors: [evidence span withheld — anchor status: external_doc]
