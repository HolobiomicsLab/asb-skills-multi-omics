---
name: differential-expression-significance-thresholding
description: Use when you have a precomputed DE table, expression/count matrix with
  metadata, or a MultiAssayExperiment object and need faceted volcano plots (log2
  fold-change vs -log10(p-value)) with thresholded significance classification.
when_to_use_negative:
- The goal is an informative heatmap with overlaid significance markers or a 2D multifeature
  tile grid rather than a volcano-style scatterplot — AnnotatedHeatmap, CompositeFeatureHeatmap,
  or the multifeature grid functions are the appropriate classes.
- Input is a pathway/feature grid table for 2D tile-and-point heatmaps (z-scores as
  tile fills, p-values as point color) rather than gene-level DE results.
- You need deep custom theming beyond the createPlot parameters (text_family, point_size,
  custom_theme) — the README defers such customization to external ComplexHeatmap
  and ggplot2 documentation.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: MultiModalGraphics
  role: Provides the ThresholdedScatterplot, ThresholdedScatterplot_table, and ThresholdedScatterplot_MAE
    S4 classes and createPlot() method that apply the significance thresholds and
    annotate per-panel counts
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: limma
  role: Runs differential expression (voom+limma) per SampleType-timePoint cell when
    the input is an expression matrix with metadata instead of a precomputed DE table
- name: MultiAssayExperiment
  role: Container for multi-omics input objects accepted by ThresholdedScatterplot_MAE
- name: dplyr
  role: 'Data wrangling: computing log2fc and negLog10p plotting variables and verifying
    per-facet thresholded counts from fold-change and p-value columns'
- name: ggplot2
  role: Underlying plotting engine; custom_theme objects (e.g. theme_minimal()) can
    be passed to createPlot for faceted volcano styling
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:b639fcc0e769384cd61943d84fe36f7775f4aa6017456f8ae4e8a69be8d75625
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
  - MultiAssayExperiment
  - MultiModalGraphics
  - R
  - dplyr
  - ggplot2
  - limma
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# differential-expression-significance-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies user-defined fold-change and p-value cutoffs to differential-expression results and renders faceted volcano-style annotated scatterplots with MultiModalGraphics' ThresholdedScatterplot class, which automatically classifies genes as upregulated, downregulated, or neutral and annotates per-panel up/down counts.

## When to use

Use when you have a precomputed DE table, expression/count matrix with metadata, or a MultiAssayExperiment object and need faceted volcano plots (log2 fold-change vs -log10(p-value)) with thresholded significance classification.

## When NOT to use

- The goal is an informative heatmap with overlaid significance markers or a 2D multifeature tile grid rather than a volcano-style scatterplot — AnnotatedHeatmap, CompositeFeatureHeatmap, or the multifeature grid functions are the appropriate classes.
- Input is a pathway/feature grid table for 2D tile-and-point heatmaps (z-scores as tile fills, p-values as point color) rather than gene-level DE results.
- You need deep custom theming beyond the createPlot parameters (text_family, point_size, custom_theme) — the README defers such customization to external ComplexHeatmap and ggplot2 documentation.

## Inputs

- Precomputed DE table (log2fc, negLog10p, regulation, SampleType, optional timePoint), e.g. extdata/scatterexample.csv from the MultiModalGraphics repo or a GSE68077-derived table
- Raw/normalized expression or count matrix (expr) with sample metadata (meta) including groupColumn, sampleType, and timepoint columns
- MultiAssayExperiment (MAE) object, e.g. miniACC with an assayName such as RNASeq2GeneNorm

## Outputs

- Faceted volcano-style ggplot scatterplot (log2fc vs -log10p) colored by up/down/neutral regulation class with per-facet up/down count annotations
- Per-panel summary counts of upregulated, downregulated, and neutral features (e.g. thresholded_scatterplot.png plus a summary table)

## How to apply

Load the DE table (e.g. GSE68077-derived extdata/scatterexample.csv from the MultiModalGraphics GitHub repo) and compute plotting variables log2fc and negLog10p from fold-change and p-value columns with dplyr. Construct a ThresholdedScatterplot object supplying highLog2fc, lowLog2fc, and negLog10pValue — for example |log2fc| > 0.585 (~1.5-fold change) and negLog10pValue > 1.301 (p < 0.05) — then call createPlot() with color1/color2/color3 for down/neutral/up classes and facet by SampleType and timePoint (e.g. T5R1, T5R10, T10R1, T10R28, T10R42). The class automatically applies the cutoffs, colors up/down/neutral points, and annotates the count of up- and down-regulated features in each facet; export via png()/pdf() or extract the ggplot object. For raw count matrices with metadata, use ThresholdedScatterplot_table, which performs variance filtering (var_quantile = 0.75) and runs voom+limma per SampleType–timePoint cell before stitching DE results into a single faceted volcano plot; for MultiAssayExperiment objects use ThresholdedScatterplot_MAE.

## Related tools

- **MultiModalGraphics** (Provides the ThresholdedScatterplot, ThresholdedScatterplot_table, and ThresholdedScatterplot_MAE S4 classes and createPlot() method that apply the significance thresholds and annotate per-panel counts) — https://github.com/famanalytics0/MultiModalGraphics
- **limma** (Runs differential expression (voom+limma) per SampleType-timePoint cell when the input is an expression matrix with metadata instead of a precomputed DE table)
- **MultiAssayExperiment** (Container for multi-omics input objects accepted by ThresholdedScatterplot_MAE)
- **dplyr** (Data wrangling: computing log2fc and negLog10p plotting variables and verifying per-facet thresholded counts from fold-change and p-value columns)
- **ggplot2** (Underlying plotting engine; custom_theme objects (e.g. theme_minimal()) can be passed to createPlot for faceted volcano styling)

## Examples

```
cs <- ThresholdedScatterplot(df_de, highLog2fc = 0.585, lowLog2fc = -0.585, negLog10pValue = 1.301); show(createPlot(cs, color1 = "tomato", color2 = "grey50", color3 = "steelblue"))
```

## Evaluation signals

- Per-facet annotated up/down counts equal the counts obtained by manually filtering with dplyr at the same cutoffs (|log2fc| > 0.585, negLog10p > 1.301).
- All points colored as 'neutral' fall inside the threshold rectangle (|log2fc| below cutoff and negLog10p below negLog10pValue).
- The negLog10pValue cutoff maps correctly to the p-value scale (1.301 corresponds to p < 0.05).
- Facets cover all expected SampleType x timePoint combinations (e.g. T5R1, T5R10, T10R1, T10R28, T10R42) with one panel each.
- Returned object is a ggplot/ThresholdedScatterplot object that can be extracted or wrapped with pdf()/png() for export.

## Limitations

- Thresholds (highLog2fc, lowLog2fc, negLog10pValue) are user-supplied, so results depend entirely on justified cutoff choices; the README example uses 1.0/-1.0/1.301 while the GSE68077 case uses 0.585/1.301.
- Advanced features and customization are not documented in the article guide; readers are deferred to external package, ComplexHeatmap, and ggplot2 documentation.
- Package and dependency version numbers are not reported in the article text (session info appears only as unevaluated code), and the vignette's claimed example content (data generation, scatterplots, heatmaps, multifeature grids) does not appear in the document body.
- In the README smoke test, the ThresholdedScatterplot constructor line must appear before createPlot, otherwise 'object cs not found' is raised.

## Evidence

- [readme] All main plotting functions let you set highLog2fc, lowLog2fc, negLog10pValue, pvalue_cutoff, fc_cutoff, and color ramps for stringency and visualization.: [evidence span withheld — anchor status: external_doc]
- [readme] A faceted volcano plot with up/down/neutral points colored as specified and counts of up and down annotated in each facet.: [evidence span withheld — anchor status: external_doc]
- [readme] The package performs variance filtering and voom+limma per SampleType-timePoint cell, then stitches DE results into a faceted volcano plot.: [evidence span withheld — anchor status: external_doc]
