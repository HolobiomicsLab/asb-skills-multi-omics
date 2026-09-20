---
name: multi-omics-statistical-annotation-overlay
description: Use when you have fold-change (z-score) and p-value matrices — or a pathway/feature
  grid table — for multi-omics or time-course data and need a publication-quality
  heatmap with embedded statistical annotations.
when_to_use_negative:
- Your goal is a faceted volcano plot (log2 fold-change vs -log10(p-value)) — use
  the package's ThresholdedScatterplot instead of a CompositeFeatureHeatmap
- Input is raw counts needing differential expression — run a limma/voom DE pipeline
  first (ThresholdedScatterplot_table handles this internally, but composite heatmaps
  expect precomputed FC/p-value matrices)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MultiModalGraphics
  role: Provides CompositeFeatureHeatmap / multifeature grid class for tile-and-point
    annotated heatmaps
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: ComplexHeatmap
  role: Heatmap rendering backend used by MultiModalGraphics for the composite heatmap
- name: dplyr
  role: Reshapes the pathway summary table into fold-change and p-value matrices indexed
    by pathway, region, and timepoint
- name: ggplot2
  role: Referenced for advanced features and customization of the plots
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:bcbcccc370fa942ca2551fc624f977becc2fcdc26add4f255d6bb871cb4af346
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# multi-omics-statistical-annotation-overlay

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Uses the MultiModalGraphics R package (CompositeFeatureHeatmap/multifeature grids) to build composite heatmaps where tile color encodes activation z-scores, overlaid dot color encodes p-value significance, and dot size encodes feature/gene counts, enabling direct quantitative comparison of multi-omics results.

## When to use

Use when you have fold-change (z-score) and p-value matrices — or a pathway/feature grid table — for multi-omics or time-course data and need a publication-quality heatmap with embedded statistical annotations.

## When NOT to use

- Your goal is a faceted volcano plot (log2 fold-change vs -log10(p-value)) — use the package's ThresholdedScatterplot instead of a CompositeFeatureHeatmap
- Input is raw counts needing differential expression — run a limma/voom DE pipeline first (ThresholdedScatterplot_table handles this internally, but composite heatmaps expect precomputed FC/p-value matrices)

## Inputs

- Fold-change (z-score) and p-value matrices
- Pathway/feature grid table (e.g. twoD_graphics.csv with activation z-scores, p-values, and gene counts per pathway per brain region and timepoint)
- Precomputed DE tables (log2fc, negLog10p, regulation, SampleType, timePoint)
- MultiAssayExperiment / SummarizedExperiment objects

## Outputs

- Composite 2D tile-and-point heatmap PNG (e.g. fig2_composite_heatmap.png) with z-score tiles, p-value dot color, and dot-sized feature counts
- sessionInfo() output for reproducibility

## How to apply

Load the summary table (e.g. twoD_graphics.csv from the MultiModalGraphics extdata) with read.csv, then reshape it with dplyr into parallel fold-change and p-value matrices indexed by pathway, condition, and timepoint. Construct a CompositeFeatureHeatmap (multifeature grid) mapping z-scores to tile fill, p-values to overlaid point color, and gene/feature counts to point size, using ComplexHeatmap as the rendering backend. Tune thresholds (highLog2fc, lowLog2fc, negLog10pValue, pvalue_cutoff, fc_cutoff) and color ramps via the customization arguments described in the README. Export the figure with png()/pdf() and report sessionInfo() for reproducibility; interpret suppressed pathways (negative z-scores, significant dot colors) as in the GSE45035 PTSD mouse-model case (inhibition of neurogenesis and synaptic plasticity pathways).

## Related tools

- **MultiModalGraphics** (Provides CompositeFeatureHeatmap / multifeature grid class for tile-and-point annotated heatmaps) — https://github.com/famanalytics0/MultiModalGraphics
- **ComplexHeatmap** (Heatmap rendering backend used by MultiModalGraphics for the composite heatmap)
- **dplyr** (Reshapes the pathway summary table into fold-change and p-value matrices indexed by pathway, region, and timepoint)
- **ggplot2** (Referenced for advanced features and customization of the plots)

## Examples

```
suppressPackageStartupMessage(library(MultiModalGraphics)); suppressPackageStartupMessage(library(ComplexHeatmap)); suppressPackageStartupMessage(library(dplyr))
```

## Evaluation signals

- Tile colors in the rendered heatmap correctly encode z-scores and overlaid dot colors encode p-value significance, dot sizes encode feature counts
- Matrices are correctly indexed by pathway × brain region (AY, HC, MPFC, SE, ST, VS) × timepoint after dplyr reshaping
- Figure exports successfully to PNG/PDF (e.g. fig2_composite_heatmap.png) and sessionInfo() is reported
- Suppressed pathways (e.g. neurogenesis and synaptic plasticity inhibition) appear as negative z-score tiles with significant dot markers
- Libraries loaded without warnings via suppressPackageStartupMessage(library(...))

## Limitations

- Advanced customization is deferred to external ComplexHeatmap and ggplot2 documentation rather than documented in the vignette
- Package and dependency version numbers are not reported in the document text; sessionInfo() appears only as unevaluated code
- No named datasets or accession identifiers are provided in the vignette itself, so users must supply their own pathway summary tables

## Evidence

- [discussion] suppressPackageStartupMessage(library(MultiModalGraphics))
  suppressPackageStartupMessage(library(ComplexHeatmap))
  suppressPackageStartupMessage(library(dplyr)): "suppressPackageStartupMessage(library(MultiModalGraphics))
  suppressPackageStartupMessage(library(ComplexHeatmap))
  suppressPackageStartupMessage(library(dplyr))"
