---
name: brain-region-timepoint-matrix-reshaping
description: Use when a long-format pathway or feature table contains one row per
  pathway × brain region × timepoint with z-score, p-value, and count columns that
  must become matrices for heatmap visualization.
when_to_use_negative:
- Input is already a pair of fold-change and p-value matrices ready for AnnotatedHeatmap
  or CompositeFeatureHeatmap
- Data is raw expression/count matrices or MultiAssayExperiment objects — use the
  package's ThresholdedScatterplot_table / MAE pipelines with limma/voom instead
- The goal is a standard single-layer heatmap without overlaid significance markers,
  where reshaping into a multifeature grid is unnecessary
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3577
tools:
- name: dplyr
  role: Reshapes the long-format pathway table into fold-change and p-value matrices
    indexed by pathway, brain region, and timepoint
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: MultiModalGraphics
  role: Consumes the reshaped matrices via CompositeFeatureHeatmap to render the 2D
    tile-and-point heatmap
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: ComplexHeatmap
  role: Underlying heatmap engine for the composite tile-and-point rendering
- name: R
  role: Runtime environment; read.csv loads twoD_graphics.csv from package extdata
    and sessionInfo() reports reproducibility
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:b93230a8838afb7cca9d79b85c9426c8a8477e2e055b6e7005d37f65267dd620
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/famanalytics0/MultiModalGraphics
  repo_url: https://github.com/famanalytics0/MultiModalGraphics
  tools:
  - ComplexHeatmap
  - MultiModalGraphics
  - R
  - dplyr
  - ggplot2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# brain-region-timepoint-matrix-reshaping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reshape a long-format pathway summary table (activation z-scores, p-values, gene counts per pathway per brain region and timepoint, e.g. GSE45035-derived twoD_graphics.csv) into paired fold-change and p-value matrices so they can feed MultiModalGraphics' CompositeFeatureHeatmap for 2D tile-and-point heatmaps.

## When to use

Use when a long-format pathway or feature table contains one row per pathway × brain region × timepoint with z-score, p-value, and count columns that must become matrices for heatmap visualization.

## When NOT to use

- Input is already a pair of fold-change and p-value matrices ready for AnnotatedHeatmap or CompositeFeatureHeatmap
- Data is raw expression/count matrices or MultiAssayExperiment objects — use the package's ThresholdedScatterplot_table / MAE pipelines with limma/voom instead
- The goal is a standard single-layer heatmap without overlaid significance markers, where reshaping into a multifeature grid is unnecessary

## Inputs

- Long-format pathway summary table (twoD_graphics.csv) with columns for activation z-score, p-value, gene count, pathway, brain region (AY, HC, MPFC, SE, ST, VS), and timepoint
- R session with MultiModalGraphics, ComplexHeatmap, and dplyr installed

## Outputs

- Fold-change (z-score) matrix with pathways as rows and region × timepoint as columns
- Aligned p-value matrix of identical dimensions
- Gene-count grid used to size overlaid significance dots
- Composite tile-and-point heatmap (e.g. fig2_composite_heatmap.png)

## How to apply

Load the pathway summary CSV (e.g. twoD_graphics.csv from the MultiModalGraphics package extdata) with read.csv, then suppressPackageStartupMessage(library(MultiModalGraphics)), library(ComplexHeatmap), and library(dplyr). Use dplyr to pivot the table into two aligned matrices indexed by pathway and by the brain regions (AY, HC, MPFC, SE, ST, VS) and timepoints: one matrix holding activation z-scores (fold-changes) and one holding p-values, keeping gene counts available as a third grid dimension. These paired matrices are exactly the input shape required by the package's CompositeFeatureHeatmap / multifeature-grid class, which maps z-scores to tile fill color, p-values to overlaid dot color, and counts to dot size. Verify the reshape by checking matrix dimension agreement (rows = pathways, columns = region × timepoint cells) and that no pathway/region/timepoint cell was dropped or duplicated before rendering to a PNG such as fig2_composite_heatmap.png and reporting sessionInfo().

## Related tools

- **dplyr** (Reshapes the long-format pathway table into fold-change and p-value matrices indexed by pathway, brain region, and timepoint) — https://github.com/famanalytics0/MultiModalGraphics
- **MultiModalGraphics** (Consumes the reshaped matrices via CompositeFeatureHeatmap to render the 2D tile-and-point heatmap) — https://github.com/famanalytics0/MultiModalGraphics
- **ComplexHeatmap** (Underlying heatmap engine for the composite tile-and-point rendering)
- **R** (Runtime environment; read.csv loads twoD_graphics.csv from package extdata and sessionInfo() reports reproducibility)

## Examples

```
twoD <- read.csv("twoD_graphics.csv"); fc <- twoD %>% tidyr::pivot_wider(names_from=c(brain_region,timepoint), values_from=z_score); p <- twoD %>% tidyr::pivot_wider(names_from=c(brain_region,timepoint), values_from=p_value)
```

## Evaluation signals

- Fold-change and p-value matrices have identical dimensions and matching row/column names (pathways vs. region × timepoint cells)
- Every pathway × region (AY, HC, MPFC, SE, ST, VS) × timepoint combination appears exactly once in the reshaped matrices — no dropped or duplicated cells
- Z-score tile colors and p-value dot colors in the rendered heatmap align with the values in the source CSV spot-checked cell-by-cell
- Dot sizes in the composite heatmap reflect the gene-count column from the original table
- sessionInfo() output is reported alongside fig2_composite_heatmap.png for reproducibility

## Limitations

- The package vignette defers advanced customization to external ComplexHeatmap and ggplot2 documentation, so non-default tile/dot styling is not covered by this skill
- No named datasets or accession identifiers (beyond the workflow's GSE45035-derived twoD_graphics.csv) are provided in the article text, so input schema must be inferred from the workflow
- Session info appears only as unevaluated code in the article, so no pinned dependency versions are available for the reshape step

## Evidence

- [other] Reshape the table with dplyr into fold-change (z-score) and p-value matrices indexed by pathway, brain region (AY, HC, MPFC, SE, ST, VS), and timepoint.: [evidence span withheld — anchor status: not_found]
- [readme] Multifeature grids: 2D tile-and-point heatmaps (z-scores as tile fills, p-values as point color, counts as point size): "**Multifeature grids**: 2D tile-and-point heatmaps (z-scores as tile fills, p-values as point color, counts as point size)"
- [intro] Informative heatmaps with overlaid significance markers from fold-change and p-value matrices: "**Informative heatmaps** with overlaid significance markers from: Fold-change & p-value matrices"
- [discussion] suppressPackageStartupMessage(library(MultiModalGraphics))
  suppressPackageStartupMessage(library(ComplexHeatmap))
  suppressPackageStartupMessage(library(dplyr)): "suppressPackageStartupMessage(library(MultiModalGraphics))
  suppressPackageStartupMessage(library(ComplexHeatmap))
  suppressPackageStartupMessage(library(dplyr))"
- [other] tile colors encoding z-scores, dot colors encoding p-value significance, and dot sizes reflecting feature counts: [evidence span withheld — anchor status: not_found]
