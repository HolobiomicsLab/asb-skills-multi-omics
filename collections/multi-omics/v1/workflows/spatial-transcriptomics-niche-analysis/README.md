# spatial-transcriptomics-niche-analysis-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **load** — spatial images and transcript coordinates to a unified persistent store  →  `spatial-omics-data-loading`, `spatial-omics-multilayer-image-alignment`, `spatial-transcriptomics-image-segmentation`
2. **segment** — transcript coordinates to resolved cell boundary polygons  →  `transcript-based-cell-segmentation`, `overlapping-patch-tiling-strategy`, `spatial-omics-patch-based-image-segmentation`, `segmentation-conflict-resolution-across-patches`, `cell-boundary-intersection-over-min-area-quantification`
3. **count_transcripts** — cell polygons and transcripts to a filtered cell-by-gene matrix  →  `spatial-transcriptomics-gene-count-aggregation`, `baysor-transcript-segmentation`
4. **quantify_channels** — retained cells and staining images to per-cell channel intensities  →  `channel-intensity-aggregation`, `polygon-chunk-spatial-intersection`, `per-cell-channel-intensity-quantification`, `cell-polygon-rasterization`
5. **niches** — aligned cell-centered image patches to niche labels and geometry tables  →  `spatial-niche-clustering-and-vectorization`, `spatial-niche-geometry-statistics`, `niche-cell-colocalization-analysis`, `cell-to-niche-group-assignment`
6. **export** — analyzed spatial store to reusable cell data and visualization files  →  `xenium-explorer-data-conversion`, `cell-boundary-polygon-export`, `pyramidal-ome-tif-image-export`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
