---
name: histogram-distribution-visualization
description: Use when a validation step requires inspecting the empirical distribution
  of a per-pair metric (e.g., IOMA segmentation-conflict scores) against a published
  pattern or threshold range.
when_to_use_negative:
- When the per-pair scores have not been computed yet — this skill only visualizes
  an existing distribution; the conflict-score computation over resolved patch polygons
  must happen first.
- When the input metric is not bounded (e.g., unnormalized overlaps), since the [0,
  1] range and the 0.07/0.8 bimodal validation pattern are specific to IOMA scores.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Sopa
  role: Generates overlapping image patches and merges per-patch Cellpose masks into
    polygons; the resulting duplicated boundary pairs feed the IOMA scores visualized
    as a histogram
  repo: https://github.com/gustaveroussy/sopa
- name: Cellpose
  role: Performs image-based segmentation on each patch (--channels DAPI --diameter
    35 --min-area 2000) whose boundary duplicates are then quantified
- name: SpatialData
  role: Unified data framework holding the toy dataset (tuto.zarr) on which patchification,
    segmentation, and conflict resolution are run
  repo: https://github.com/scverse/spatialdata
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:ef06ed3041d64754dc7041e958339eee86f37101911cdeb03a3c051a50d0f7db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - Cellpose
  - Sopa
  - SpatialData
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# histogram-distribution-visualization

## Summary

Plot a histogram of a computed score distribution (e.g., intersection-over-min-area (IOMA) conflict scores of cell boundaries across overlapping patches) to verify it reproduces a reported bimodal pattern with values bounded in [0, 1].

## When to use

Use when a validation step requires inspecting the empirical distribution of a per-pair metric (e.g., IOMA segmentation-conflict scores) against a published pattern or threshold range.

## When NOT to use

- When the per-pair scores have not been computed yet — this skill only visualizes an existing distribution; the conflict-score computation over resolved patch polygons must happen first.
- When the input metric is not bounded (e.g., unnormalized overlaps), since the [0, 1] range and the 0.07/0.8 bimodal validation pattern are specific to IOMA scores.

## Inputs

- ioma_scores.csv containing one intersection-over-min-area (IOMA) score per pair of duplicated cell boundaries across adjacent patches
- Segmentation polygons merged per patch from Cellpose masks on a SpatialData (tuto.zarr) object

## Outputs

- ioma_conflict_distribution.png — histogram of IOMA scores

## How to apply

After computing per-pair conflict scores — here, the IOMA between duplicated cell boundaries across adjacent overlapping patches generated with `sopa patchify image tuto.zarr --patch-width-pixel 1500 --patch-overlap-pixel 50` and merged via `sopa resolve cellpose tuto.zarr` — export all scores to ioma_scores.csv. Load the CSV, plot a histogram of the IOMA values, and save it as ioma_conflict_distribution.png. Judge correctness by checking that the distribution is bimodal with most conflicts falling below 0.07 (same cell segmented twice) or above 0.8 (distinct cells slightly overlapping), matching Fig. 2e / Supplementary Table 1, and that all scores lie within [0, 1].

## Related tools

- **Sopa** (Generates overlapping image patches and merges per-patch Cellpose masks into polygons; the resulting duplicated boundary pairs feed the IOMA scores visualized as a histogram) — https://github.com/gustaveroussy/sopa
- **Cellpose** (Performs image-based segmentation on each patch (--channels DAPI --diameter 35 --min-area 2000) whose boundary duplicates are then quantified)
- **SpatialData** (Unified data framework holding the toy dataset (tuto.zarr) on which patchification, segmentation, and conflict resolution are run) — https://github.com/scverse/spatialdata

## Examples

```
import pandas as pd, matplotlib.pyplot as plt; scores = pd.read_csv('ioma_scores.csv'); plt.hist(scores['ioma'], bins=50, range=(0,1)); plt.xlabel('IOMA'); plt.savefig('ioma_conflict_distribution.png')
```

## Evaluation signals

- The histogram shows a bimodal distribution: most IOMA scores below 0.07 or above 0.8, reproducing the reported pattern (Fig. 2e, Supplementary Table 1).
- All IOMA scores lie within the valid interval [0, 1].
- ioma_conflict_distribution.png is produced alongside ioma_scores.csv, with the histogram consistent with the exported score values.
- The number of conflict pairs matches the duplicated boundaries found after merging the 4 patches of the toy dataset.

## Limitations

- The validation pattern (scores below 0.07 or above 0.8) was established on a toy dataset with a 50-pixel patch overlap; other patch sizes, overlaps, or real tissues may shift the distribution.
- The histogram validates conflict resolution only indirectly — it does not itself measure final segmentation quality on full-image segmentation.

## Evidence

- [results] Patch-based segmentation does not impact segmentation quality because most conflicting cell boundaries have an intersection-over-min-area (IOMA) lower than 0.07 or higher than 0.8: [evidence span withheld — anchor status: not_found]
- [methods] Plot a histogram of the IOMA scores and save it as ioma_conflict_distribution.png: [evidence span withheld — anchor status: not_found]
- [intro] Generate overlapping patches of images and/or transcripts: "we generate overlapping patches of images and/or transcripts. Segmentation can then be performed for each individual patch"
- [intro] Convert cell segmentation masks into polygons and merge over patches to resolve conflicts: "the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts."
