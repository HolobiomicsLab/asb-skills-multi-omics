---
name: baysor-transcript-segmentation
description: Use when you have a SpatialData object with a transcript dataframe from
  imaging-based spatial transcriptomics (e.g. Xenium, MERSCOPE, CosMX) and need transcript-based
  cell segmentation that fits in memory.
when_to_use_negative:
- The input is image-based staining data needing image segmentation — use Cellpose
  (image-based) instead; Baysor is for transcripts-based segmentation.
- You need segmentation directly on already-segmented cell data — Sopa's segmentation
  step assumes unsegmented transcript coordinates.
- Proseg is preferred and available independently — the Sopa methods note Proseg can
  be installed as an alternative transcript segmenter.
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3308
tools:
- name: Sopa
  role: Pipeline that patchifies the transcript dataframe, launches Baysor per patch,
    and resolves cell polygons across patch boundaries
  repo: https://github.com/gustaveroussy/sopa
- name: Baysor
  role: Bayesian transcripts-based segmentation engine run inside each spatial-transcriptomics
    patch
  repo: https://github.com/kharchenkolab/Baysor
- name: SpatialData
  role: Unified data framework holding the transcript dataframe and image that Sopa
    operates on
  repo: https://github.com/scverse/spatialdata
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:08b565211b86aa0be5f609751d47e0266772d8203d6828cba0ed0f8f7842a505
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - Baysor
  - MERSCOPE
  - Sopa
  - SpatialData
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# baysor-transcript-segmentation

## Summary

Run Baysor, a Bayesian segmentation method for imaging-based spatial transcriptomics, on transcript-level patches within the Sopa pipeline so that cell segmentation scales to millions of cells with 10-100x lower RAM and up to 100x faster runtime than whole-transcript-set segmentation.

## When to use

Use when you have a SpatialData object with a transcript dataframe from imaging-based spatial transcriptomics (e.g. Xenium, MERSCOPE, CosMX) and need transcript-based cell segmentation that fits in memory.

## When NOT to use

- The input is image-based staining data needing image segmentation — use Cellpose (image-based) instead; Baysor is for transcripts-based segmentation.
- You need segmentation directly on already-segmented cell data — Sopa's segmentation step assumes unsegmented transcript coordinates.
- Proseg is preferred and available independently — the Sopa methods note Proseg can be installed as an alternative transcript segmenter.

## Inputs

- Imaging-based spatial transcriptomics raw output (Xenium, MERSCOPE, CosMX) loaded as a SpatialData object with a transcript dataframe (x, y, gene columns)
- Baysor config.toml (scale=6 microns, min_molecules_per_cell=10)
- Optional transcript filters: sopa.settings.gene_exclude_pattern and a 'low_quality_transcript' boolean column in the transcript dataframe

## Outputs

- Per-patch Baysor cell segmentation results on transcript patches
- Merged cell polygons after 'sopa resolve baysor' conflict resolution across patch boundaries
- Cell-by-transcript counts for downstream aggregation/annotation
- Peak RAM and wall-clock measurements per run (for benchmark workflows, e.g. written to benchmark_results.csv)

## How to apply

Convert the raw technology output into a SpatialData object with a Sopa reader (e.g. sopa.io.merscope, sopa.io.xenium), then run 'sopa patchify transcripts' with overlapping patches (e.g. --patch-width-microns 200) so each Baysor job only sees a small spatial region. Configure Baysor per patch (config.toml with scale=6 microns and min_molecules_per_cell=10); optionally pre-filter transcripts using sopa.settings.gene_exclude_pattern or a low_quality_transcript boolean column so low-confidence rows are not segmented. After per-patch segmentation, run 'sopa resolve baysor' to convert the per-patch cell masks into polygons and merge them across patch boundaries, removing duplicate/split cells at overlaps. The patch-overlap strategy exists because Baysor on the whole transcript set demands high-performance clusters with substantial memory, while per-patch execution drastically reduces peak RAM and wall-clock time.

## Related tools

- **Sopa** (Pipeline that patchifies the transcript dataframe, launches Baysor per patch, and resolves cell polygons across patch boundaries) — https://github.com/gustaveroussy/sopa
- **Baysor** (Bayesian transcripts-based segmentation engine run inside each spatial-transcriptomics patch) — https://github.com/kharchenkolab/Baysor
- **SpatialData** (Unified data framework holding the transcript dataframe and image that Sopa operates on) — https://github.com/scverse/spatialdata

## Examples

```
sopa patchify transcripts ./data --patch-width-microns 200 && sopa resolve baysor
```

## Evaluation signals

- benchmark_results.csv exists and contains memory-ratio and time-ratio entries for Baysor on each dataset, with the patched run showing substantially lower peak RAM than the unpatched whole-transcript baseline
- 'sopa resolve baysor' completes without leaving duplicate or split cells at patch boundaries, confirming polygon merging removed patch artefacts
- Cells below min_molecules_per_cell=10 transcripts are excluded from the segmentation output
- All transcript rows flagged low_quality_transcript=True or matching the gene_exclude_pattern were not used during segmentation
- Resulting cell polygons are stored in the SpatialData object and counts are aggregatable per cell for annotation

## Limitations

- Baysor must be installed independently (the cpp branch is a port preserving the core algorithm of the v0.7.1 release line), and availability of future validated methods is contingent on publication
- Current spatial technologies provide either proteins or transcripts; without an extra information layer (e.g. a protein-staining layer in the Xenium pancreatic cancer data), some cell populations (B cells) cannot be extracted
- Patch-based segmentation introduces boundary artefacts that require the resolve/merge step; min_area, min_transcripts and min_intensity_ratio thresholds may remove small or low-signal cells

## Evidence

- [intro] Segmentation can then be performed for each individual patch, and we currently support Cellpose18 (image-based segmentation) and Baysor19 (transcripts-based segmentation).: "Segmentation can then be performed for each individual patch, and we currently support Cellpose18 (image-based segmentation) and Baysor19 (transcripts-based segmentation)."
- [readme] Baysor segments imaging-based spatial transcriptomics data using spatial position, local gene composition, and optional prior segmentation masks.: [evidence span withheld — anchor status: not_found]
- [methods] create a (boolean) column called "low_quality_transcript" to your transcript dataframe. The rows whose value is True will not be used during segmentation.: "create a (boolean) column called "low_quality_transcript" to your transcript dataframe. The rows whose value is True will not be used during segmentation."
- [intro] the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts.: "the cell segmentation masks are converted into polygons and merged over all patches to remove potential artefacts."
