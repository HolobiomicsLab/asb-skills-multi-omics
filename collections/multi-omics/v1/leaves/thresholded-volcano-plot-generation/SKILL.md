---
name: thresholded-volcano-plot-generation
description: Use when you have a precomputed DE table (log2fc, negLog10p, regulation,
  SampleType, optional timePoint) and need publication-quality faceted volcano plots
  with threshold-based coloring and per-panel up/down counts.
when_to_use_negative:
- You need a heatmap with significance markers or a 2D multifeature grid — use AnnotatedHeatmap,
  CompositeFeatureHeatmap, or the multifeature-grid class instead
- Your input is a MultiAssayExperiment object — use ThresholdedScatterplot_MAE() rather
  than the plain-table class
- Only a single flat plot without faceting or threshold annotation is needed and a
  lightweight ggplot2 volcano would suffice
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: MultiModalGraphics
  role: Provides the ThresholdedScatterplot S4 class and createPlot()/show() methods
    that apply fold-change and p-value cutoffs and annotate per-facet up/down counts
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: ggplot2
  role: Underlying plotting engine for the faceted volcano plot; used for custom themes
    via custom_theme (e.g., theme_minimal())
- name: dplyr
  role: Data wrangling of DE tables (computing log2fc and negLog10p columns) before
    plot construction
- name: limma
  role: Differential-expression engine run under the hood by ThresholdedScatterplot_table
    when starting from expression matrix + metadata
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:f82dd188a0edbed07c90f173e7b6071b4c1c975dbfe40aa31d1394b8d1ba3ad2
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
  - MultiModalGraphics
  - R
  - dplyr
  - ggplot2
  - limma
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.2727272727272727
n_claims: 11
---

# thresholded-volcano-plot-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate faceted volcano-style thresholded scatterplots (log2 fold-change vs. -log10(p-value)) from precomputed differential-expression tables using the MultiModalGraphics ThresholdedScatterplot S4 class, which automatically classifies features as up-, down-, or non-significant and annotates per-facet counts.

## When to use

Use when you have a precomputed DE table (log2fc, negLog10p, regulation, SampleType, optional timePoint) and need publication-quality faceted volcano plots with threshold-based coloring and per-panel up/down counts.

## When NOT to use

- You need a heatmap with significance markers or a 2D multifeature grid — use AnnotatedHeatmap, CompositeFeatureHeatmap, or the multifeature-grid class instead
- Your input is a MultiAssayExperiment object — use ThresholdedScatterplot_MAE() rather than the plain-table class
- Only a single flat plot without faceting or threshold annotation is needed and a lightweight ggplot2 volcano would suffice

## Inputs

- Precomputed DE data.frame with columns log2fc, negLog10p, regulation (up/down), SampleType, and optional timePoint
- Raw/normalized expression matrix + sample metadata (for ThresholdedScatterplot_table; counts auto-detected via dataType='auto')
- Optional: GEO-derived differential-expression table (e.g., GSE68077) with fold-change and p-value columns

## Outputs

- Faceted volcano ggplot (log2fc vs. negLog10p) with up/down/neutral points colored per user colors and per-facet counts of up- and down-regulated features annotated
- Exported figure file (e.g., thresholded_scatterplot.png) via pdf()/png() wrappers
- Per-panel up/down count summary table

## How to apply

Load the DE table (e.g., extdata/scatterexample.csv from the famanalytics0/MultiModalGraphics repo, or a GEO GSE68077-derived table) into R and compute log2 fold-change and -log10(p-value) with base R or dplyr. Construct a ThresholdedScatterplot object, setting highLog2fc, lowLog2fc, and negLog10pValue cutoffs (e.g., 1.0 / -1.0 / 1.301, or |log2fc| > 0.585 with negLog10p > 1.301 for a ~1.5-fold, p < 0.05 cutoff). Call createPlot() with color1/color2/color3 (down/neutral/up), then show() to draw the faceted ggplot, one panel per SampleType-timePoint cell. For raw count matrices plus metadata, use ThresholdedScatterplot_table() instead, which runs voom+limma per SampleType-timePoint cell under the hood and applies a variance filter (var_quantile = 0.75). The class automatically annotates the number of upregulated and downregulated features within each facet, which is the key output to verify.

## Related tools

- **MultiModalGraphics** (Provides the ThresholdedScatterplot S4 class and createPlot()/show() methods that apply fold-change and p-value cutoffs and annotate per-facet up/down counts) — https://github.com/famanalytics0/MultiModalGraphics
- **ggplot2** (Underlying plotting engine for the faceted volcano plot; used for custom themes via custom_theme (e.g., theme_minimal()))
- **dplyr** (Data wrangling of DE tables (computing log2fc and negLog10p columns) before plot construction)
- **limma** (Differential-expression engine run under the hood by ThresholdedScatterplot_table when starting from expression matrix + metadata)

## Examples

```
cs1 <- ThresholdedScatterplot(df_de, highLog2fc=1.0, lowLog2fc=-1.0, negLog10pValue=1.301); cs1 <- createPlot(cs1, color1="tomato", color2="grey50", color3="steelblue", title="Example Volcano"); show(cs1)
```

## Evaluation signals

- Each facet contains an annotation with counts of 'up' and 'down' features matching counts recomputed from the thresholds (|log2fc| > highLog2fc/lowLog2fc and negLog10p > negLog10pValue)
- Point colors match the user-supplied color1/color2/color3 mapping to down/neutral/up categories
- The plot is faceted by SampleType (X) and timePoint (Y), producing one panel per tissue/region/timepoint cell (e.g., T5R1, T5R10, T10R1, T10R28, T10R42)
- Axis labels reflect the customization arguments (xlab, ylab) and the rendering succeeds via show(createPlot(cs1, ...)) without object-not-found errors

## Limitations

- The plain ThresholdedScatterplot requires a precomputed DE table with specific columns (log2fc, negLog10p, regulation, SampleType); other inputs need ThresholdedScatterplot_table or _MAE variants
- For matrix+metadata input, the lowest-variance 25% of genes are dropped by default (var_quantile = 0.75), which changes the feature set before DE
- For advanced customization the user is deferred to external ggplot2/ComplexHeatmap documentation; no changelog or dependency versions are documented

## Evidence

- [readme] README facet count annotation: [evidence span withheld — anchor status: external_doc]
- [readme] README matrix+metadata pipeline: [evidence span withheld — anchor status: external_doc]
