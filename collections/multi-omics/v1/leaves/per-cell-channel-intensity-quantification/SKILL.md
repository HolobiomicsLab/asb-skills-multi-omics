---
name: per-cell-channel-intensity-quantification
description: Use when image-based spatial omics data (Xenium, CosMX, MACSima, PhenoCycler)
  contains protein-staining channels and per-cell intensity values are needed for
  cell-type annotation or niche enrichment analysis.
when_to_use_negative:
- Input has only transcript data without any staining channels — intensity quantification
  is not applicable; transcript counting alone (sopa.aggregate min_transcripts) should
  be used instead.
- Cells have not yet been segmented or masks have not been merged across patches —
  aggregation would average intensities over invalid boundaries.
- The dataset is already a per-cell feature/intensity table (e.g. proseg or Baysor
  output with expression matrix) — no re-aggregation is needed.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Sopa
  role: Performs the aggregation step averaging staining intensities and counting
    transcripts inside each segmented cell; provides min_intensity_ratio filtering
  repo: https://github.com/gustaveroussy/sopa
- name: SpatialData
  role: Unified framework holding images, segmentation polygons, and aggregated per-cell
    values for downstream analysis
  repo: https://github.com/scverse/spatialdata
- name: Xenium
  role: Source technology providing the pancreatic cancer dataset whose protein-staining
    channels (incl. TROP2) are aggregated per cell
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48981-z
    title: 'Sopa: a technology-invariant pipeline for analyses of image-based spatial
      omics'
  source_content_hash: sha256:0ca69588ad225c67ac296a3bb6d899da2af4beccfda79dde32cb259699035e79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gustaveroussy/sopa
  tools:
  - MERSCOPE
  - Sopa
  - SpatialData
  - Xenium
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# per-cell-channel-intensity-quantification

## Summary

Quantifies staining-channel intensities (e.g. TROP2 in the Xenium pancreatic cancer dataset) inside each segmented cell boundary using Sopa's aggregation step, producing a cell-by-channel intensity table that supports staining-based annotation and niche-level characterization.

## When to use

Use when image-based spatial omics data (Xenium, CosMX, MACSima, PhenoCycler) contains protein-staining channels and per-cell intensity values are needed for cell-type annotation or niche enrichment analysis.

## When NOT to use

- Input has only transcript data without any staining channels — intensity quantification is not applicable; transcript counting alone (sopa.aggregate min_transcripts) should be used instead.
- Cells have not yet been segmented or masks have not been merged across patches — aggregation would average intensities over invalid boundaries.
- The dataset is already a per-cell feature/intensity table (e.g. proseg or Baysor output with expression matrix) — no re-aggregation is needed.

## Inputs

- SpatialData object with cell segmentation polygons/boundaries
- Multi-channel staining image (e.g. protein channels of the Xenium pancreatic cancer run, including TROP2)
- Segmentation masks converted to polygons and merged over patches
- Optional min_intensity_ratio threshold for low-intensity cell filtering

## Outputs

- Per-cell average intensity table (cell x channel), including TROP2, stored in the SpatialData object / AnnData table
- Per-niche intensity distributions (e.g. TROP2 intensity per H&E-derived niche) written to niche_enrichment_trop2.csv
- Filtered cell set with low-intensity cells removed

## How to apply

After cell segmentation masks are converted into polygons and merged across patches, run Sopa's aggregation step (sopa.aggregate) over the SpatialData object to average the staining intensities of each channel inside each cell while simultaneously counting transcripts per cell. Apply the min_intensity_ratio argument to remove cells with low fluorescence intensity, mirroring the min_transcripts filter for transcript counts. In the Xenium pancreatic cancer H&E-niche workflow, the aggregated per-cell intensities (including the TROP2 channel) are joined to niche-assigned cells so that per-niche TROP2 intensity distributions can be computed (e.g. TROP2 more expressed in tumour-specific niches 0 and 3) and written to niche_enrichment_trop2.csv. Because Sopa is built on SpatialData, aggregation results are stored in the unified SpatialData object, making them directly usable for downstream annotation and geometric/spatial analysis.

## Related tools

- **Sopa** (Performs the aggregation step averaging staining intensities and counting transcripts inside each segmented cell; provides min_intensity_ratio filtering) — https://github.com/gustaveroussy/sopa
- **SpatialData** (Unified framework holding images, segmentation polygons, and aggregated per-cell values for downstream analysis) — https://github.com/scverse/spatialdata
- **Xenium** (Source technology providing the pancreatic cancer dataset whose protein-staining channels (incl. TROP2) are aggregated per cell)

## Examples

```
import sopa; sopa.aggregate(sdata, min_intensity_ratio=0.2)  # average TROP2 and other channel intensities inside each cell
```

## Evaluation signals

- Every segmented cell has a finite intensity value for each staining channel (no NaNs) after aggregation.
- Cells failing the min_intensity_ratio threshold are removed; final cell count matches the expected N (175,022 in the Xenium H&E-niche task).
- Per-niche TROP2 distributions reproduce the expected pattern: higher TROP2 in tumour-specific niches 0 and 3 relative to other niches.
- Aggregated table is stored in the SpatialData object and aligns 1:1 with cell polygons and transcript counts.

## Limitations

- B cell extraction in the Xenium data depended on the availability of the protein-staining layer; without such a layer, intensity-based populations cannot be recovered.
- Current spatial technologies provide either proteins or transcripts; extra information layers are a future development, so some technologies yield no channels to aggregate.
- Aggregation quality is bounded by segmentation quality — boundaries from transcript-only methods (Baysor, Proseg) can be unreliable for cells with very few transcripts.

## Evidence

- [intro] Aggregate: average staining intensities and count transcripts inside each cell: "we average the staining intensities and count the transcripts inside each cell, allowing further tasks such as annotation."
- [methods] min_intensity_ratio cell filtering: "A [min_intensity_ratio](..//api/aggregation/#sopa.aggregate) value to remove cells with a low fluorescence intensity"
- [discussion] B cell extraction depended on the protein layer: "Without this protein layer, extracting the B cell population in the Xenium data would not have been possible."
- [abstract] Technology coverage of Sopa: "diverse spatial omics technologies, such as Xenium, MERSCOPE, CosMX in spatial-transcriptomics, and MACSima and PhenoCycler in multiplex imaging"
