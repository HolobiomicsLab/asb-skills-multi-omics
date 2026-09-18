---
name: faceted-multi-panel-visualization
description: Use when the goal is to visualize differential-expression results as
  faceted, thresholded volcano/scatter panels from precomputed DE tables, expression
  matrices + metadata, or MultiAssayExperiment objects, with per-panel significance
  counts.
when_to_use_negative:
- Input is a fold-change/p-value matrix for heatmap-style significance markers — use
  the AnnotatedHeatmap / CompositeFeatureHeatmap classes instead of a faceted volcano
- Input is a raw expression matrix without group/sample metadata columns — ThresholdedScatterplot_table
  requires meta with groupColumn, sampleType, and timepoint to run limma DE per cell
- Only a single simple x-y scatter is needed without thresholds, facets, or significance
  annotation — the ThresholdedScatterplot machinery (cutoffs and per-facet counts)
  adds unnecessary complexity
edam_operation: http://edamontology.org/operation_0337
tools:
- name: MultiModalGraphics
  role: Primary package providing the ThresholdedScatterplot / ThresholdedScatterplot_table
    / ThresholdedScatterplot_MAE classes and createPlot() for faceted volcano plots
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: ggplot2
  role: Underlying rendering engine for the faceted scatterplots; themes and plot
    customization are delegated to ggplot2 documentation
- name: limma
  role: Runs differential expression (voom+limma) under the hood for ThresholdedScatterplot_table
    / _MAE when starting from matrices or MAE objects
- name: dplyr
  role: Data wrangling of DE tables (computing log2fc and -log10p columns) before
    plotting
- name: ComplexHeatmap
  role: Companion package for the heatmap side of the package; loaded alongside MultiModalGraphics
    in the vignette workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:6408b9292ffca03aa379a8f8778ac10c1555954f953358ab8fef3347b597c75f
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
  - limma
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# faceted-multi-panel-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate publication-quality faceted volcano plots (log2 fold-change vs. -log10 p-value) and annotated multi-panel scatterplots using the MultiModalGraphics R package's ThresholdedScatterplot class, which automatically applies fold-change/p-value cutoffs and annotates up/down/neutral feature counts per facet.

## When to use

Use when the goal is to visualize differential-expression results as faceted, thresholded volcano/scatter panels from precomputed DE tables, expression matrices + metadata, or MultiAssayExperiment objects, with per-panel significance counts.

## When NOT to use

- Input is a fold-change/p-value matrix for heatmap-style significance markers — use the AnnotatedHeatmap / CompositeFeatureHeatmap classes instead of a faceted volcano
- Input is a raw expression matrix without group/sample metadata columns — ThresholdedScatterplot_table requires meta with groupColumn, sampleType, and timepoint to run limma DE per cell
- Only a single simple x-y scatter is needed without thresholds, facets, or significance annotation — the ThresholdedScatterplot machinery (cutoffs and per-facet counts) adds unnecessary complexity

## Inputs

- Precomputed DE table with columns log2fc, negLog10p, regulation, SampleType, timePoint (e.g., GSE68077-derived scatterexample.csv)
- Expression/count matrix (rows = genes, columns = samples) plus sample metadata (Group, SampleType, timePoint)
- MultiAssayExperiment (MAE) object, e.g., miniACC with RNASeq2GeneNorm assay

## Outputs

- Faceted multi-panel volcano/scatterplot (ggplot object) with up/down/neutral points colored by threshold and counts of up and down genes annotated per facet
- Exported figure file (e.g., thresholded_scatterplot.png) and optionally a summary table of per-panel up/down counts

## How to apply

Load the differential-expression table (e.g., extdata/scatterexample.csv from the MultiModalGraphics GitHub repo, derived from GEO accession GSE68077) and compute log2 fold-change and -log10(p-value). Construct a ThresholdedScatterplot object with explicit thresholds such as highLog2fc = 1.0, lowLog2fc = -1.0 (or |log2fc| > 0.585 for ~1.5-fold change) and negLog10pValue = 1.301 (p < 0.05); the class classifies features as up, down, or neutral. Call createPlot() supplying colors for down/neutral/up (color1/color2/color3), axis labels, text size, point size/alpha, legend options, and an optional custom_theme such as theme_minimal(); facets are laid out by SampleType (X) and timePoint (Y), e.g., T5R1, T5R10, T10R1, T10R28, T10R42. For expression matrices + metadata, use ThresholdedScatterplot_table with groupColumn/sampleType/timepoint, dataType = "auto", and var_quantile (e.g., 0.75) to run variance filtering plus voom+limma per SampleType–timePoint cell before plotting. Show the plot with show() and export via png()/pdf(); correctness is judged by the presence of per-facet up/down count annotations and expected threshold-based coloring.

## Related tools

- **MultiModalGraphics** (Primary package providing the ThresholdedScatterplot / ThresholdedScatterplot_table / ThresholdedScatterplot_MAE classes and createPlot() for faceted volcano plots) — https://github.com/famanalytics0/MultiModalGraphics
- **ggplot2** (Underlying rendering engine for the faceted scatterplots; themes and plot customization are delegated to ggplot2 documentation)
- **limma** (Runs differential expression (voom+limma) under the hood for ThresholdedScatterplot_table / _MAE when starting from matrices or MAE objects)
- **dplyr** (Data wrangling of DE tables (computing log2fc and -log10p columns) before plotting)
- **ComplexHeatmap** (Companion package for the heatmap side of the package; loaded alongside MultiModalGraphics in the vignette workflow)

## Examples

```
cs1 <- ThresholdedScatterplot(data = df_de, highLog2fc = 1.0, lowLog2fc = -1.0, negLog10pValue = 1.301); cs1 <- createPlot(cs1, color1 = "tomato", color2 = "grey50", color3 = "steelblue"); show(cs1)
```

## Evaluation signals

- Each facet shows annotations of the counts of 'up' and 'down' genes passing the user-defined thresholds
- Point colors match the color1/color2/color3 specification for down/neutral/up categories as documented in the README example
- Points fall on the expected axes: x = log2 fold-change, y = -log10(p-value), with facets split by SampleType (X) and timePoint (Y)
- Genes classified as up satisfy log2fc > highLog2fc and -log10p > negLog10pValue (e.g., 1.0 and 1.301, or |log2fc| > 0.585 with p < 0.05)
- The ggplot object can be extracted and exported with png()/pdf() and reproduced in RMarkdown without hard-coded numeric flags in the plotting call

## Limitations

- Advanced features and customization are not documented in the article guide; readers are deferred to external ComplexHeatmap and ggplot2 documentation
- The vignette claims to include examples (data generation, scatterplots, informative heatmaps, multifeature grids) but no such example content appears in the document body, and no named datasets or accession identifiers are provided there
- No changelog is available and package/dependency version numbers are not reported in the document text; session info appears only as unevaluated code (README reports version 0.99.5)

## Evidence

- [readme] README result statement: faceted volcano plot with up/down/neutral coloring and counts annotated in each facet: [evidence span withheld — anchor status: external_doc]
- [readme] README customization tips naming threshold parameters highLog2fc, lowLog2fc, negLog10pValue, pvalue_cutoff, fc_cutoff: [evidence span withheld — anchor status: external_doc]
