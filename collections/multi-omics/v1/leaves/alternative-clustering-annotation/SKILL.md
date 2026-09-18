---
name: alternative-clustering-annotation
description: Use when a preprocessed cell-by-gene AnnData table from SPArrOW's tb.preprocess_transcriptomics
  needs a second, centroid-based clustering (tb.kmeans) stored alongside the existing
  Leiden labels for comparison.
when_to_use_negative:
- Input is raw image or unallocated transcript data — clustering operates on the preprocessed
  cell-by-gene table, which only exists after the im.* steps, tb.allocate/tb.bin_counts,
  and tb.preprocess_transcriptomics
- Only the primary graph-based clustering is needed — tb.leiden alone on the preprocessed
  table suffices; tb.kmeans is specifically for adding a centroid-based alternative
  partition
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SPArrOW
  role: Provides the tb.kmeans table operation that assigns k-means cluster labels
    to cells of the allocated cell-by-gene table, paired with tb.leiden in the clustering
    stage
  repo: https://github.com/saeyslab/napari-sparrow
- name: napari-sparrow
  role: Package/Napari plugin through which SPArrOW table operations are run and whose
    GUI is used to tune clustering parameters on small image crops before the full
    pipeline run
  repo: https://github.com/saeyslab/napari-sparrow
- name: Python
  role: Runtime requirement (Python 3.11 or newer) for executing the SPArrOW pipeline
    that performs the k-means clustering and .h5ad serialization
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.07.04.601829
    title: 'SPArrOW: a flexible, interactive and scalable pipeline for spatial transcriptomics
      analysis'
  source_content_hash: sha256:d7d32edcd83c9ab7b558090b475957231045d5354604fa5ab90e132a756d5575
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
  - napari-sparrow
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.18181818181818182
n_claims: 11
---

# alternative-clustering-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs SPArrOW's tb.kmeans table operation on the allocated cell-by-gene AnnData table to give every cell an alternative k-means cluster label, stored alongside the primary tb.leiden clustering so the two partitions can be compared on the same spatial transcriptomics data.

## When to use

Use when a preprocessed cell-by-gene AnnData table from SPArrOW's tb.preprocess_transcriptomics needs a second, centroid-based clustering (tb.kmeans) stored alongside the existing Leiden labels for comparison.

## When NOT to use

- Input is raw image or unallocated transcript data — clustering operates on the preprocessed cell-by-gene table, which only exists after the im.* steps, tb.allocate/tb.bin_counts, and tb.preprocess_transcriptomics
- Only the primary graph-based clustering is needed — tb.leiden alone on the preprocessed table suffices; tb.kmeans is specifically for adding a centroid-based alternative partition

## Inputs

- Preprocessed cell-by-gene AnnData table produced by tb.preprocess_transcriptomics (from tb.allocate/tb.bin_counts transcript allocations)
- SPArrOW/Hydra configuration (.yaml) selecting the clustering steps, e.g. tb.kmeans alongside tb.leiden

## Outputs

- AnnData object with k-means cluster labels stored as an alternative clustering to tb.leiden
- .h5ad output file containing the annotated cell-by-gene table with k-means assignments, plus the per-step configuration .yaml

## How to apply

Load the preprocessed cell-by-gene AnnData table emitted by tb.preprocess_transcriptomics (itself fed by tb.allocate/tb.bin_counts transcript allocation) and run the SPArrOW table operation tb.kmeans on it; SPArrOW's clustering stage pairs tb.leiden and tb.kmeans, so k-means is computed on exactly the same input that produced the Leiden clustering. Choose the cluster count k explicitly — the article gives no default — and tune it with the napari-sparrow plugin, which allows parameter tuning on small crops of the image before a full run. Store the k-means assignments as an alternative clustering in the AnnData object (e.g., a separate obs column) rather than overwriting the Leiden labels. Save the annotated object to an .h5ad output file; SPArrOW also saves a corresponding configuration .yaml file after every step, so a k-means config yaml should appear next to the .h5ad. Validate that the output .h5ad exists, carries a k-means assignment for every cell, and still contains the Leiden clustering for side-by-side comparison.

## Related tools

- **SPArrOW** (Provides the tb.kmeans table operation that assigns k-means cluster labels to cells of the allocated cell-by-gene table, paired with tb.leiden in the clustering stage) — https://github.com/saeyslab/napari-sparrow
- **napari-sparrow** (Package/Napari plugin through which SPArrOW table operations are run and whose GUI is used to tune clustering parameters on small image crops before the full pipeline run) — https://github.com/saeyslab/napari-sparrow
- **Python** (Runtime requirement (Python 3.11 or newer) for executing the SPArrOW pipeline that performs the k-means clustering and .h5ad serialization)

## Examples

```
uv run sparrow "steps=[tb.preprocess_transcriptomics, tb.leiden, tb.kmeans]"
```

## Evaluation signals

- Output .h5ad file exists and loads as an AnnData object with the same number of cells as the input cell-by-gene table
- Every cell carries a k-means cluster assignment — no missing/NaN labels — forming a small categorical set consistent with the chosen k
- The tb.leiden clustering computed from the same input is still present next to the k-means labels (alternative clustering stored, not overwritten)
- A configuration .yaml file for the k-means step is written to the chosen output directory, matching SPArrOW's per-step config saving
- Clustering was run on the tb.preprocess_transcriptomics output (post transcript allocation), matching the card's specified input

## Limitations

- The package is still under very active development (README), so tb.kmeans API and config details may change between versions
- The article provides no default cluster count or parameter guidance for tb.kmeans; k must be chosen by the user (tunable via the Napari plugin on small crops)
- The pipeline requires Python 3.11 or newer and uses uv for environment/dependency management; older environments may not run the table operations

## Evidence

- [methods] SPArrOW is a library for single-cell Spatial Transcriptomics data analysis.: "SPArrOW is a library for single-cell Spatial Transcriptomics data analysis."
- [methods] tb.leiden
    tb.kmeans: "tb.leiden
    tb.kmeans"
- [methods] After every step, a corresponding configuration _.yaml_ file will be saved in the output directory chosen by the user.: "After every step, a corresponding configuration _.yaml_ file will be saved in the output directory chosen by the user."
- [methods] Use the plugin to tune the parameters of `SPArrOW` for the different steps of the pipeline. Tuning can be done on small crops of the image.: "Use the plugin to tune the parameters of `SPArrOW` for the different steps of the pipeline. Tuning can be done on small crops of the image."
- [methods] The project requires Python 3.11 or newer.: "The project requires Python 3.11 or newer."
- [readme] Note: This package is still under very active development.: "Note: This package is still under very active development."
