---
name: volcano-plot-threshold-annotation
description: Use when rendering faceted volcano plots (log2 fold-change vs. -log10
  p-value) from DE tables, expression matrices + metadata, or MultiAssayExperiment
  objects and you need per-facet up/down/neutral counts annotated at explicit thresholds.
when_to_use_negative:
- The goal is an annotated heatmap with significance markers rather than a volcano
  plot — use AnnotatedHeatmap or CompositeFeatureHeatmap from the same package instead
- Only raw expression data without group/sample metadata is available and no differential-expression
  pipeline can be run, since the volcano requires per-feature fold-change and p-value
  values
- Advanced customization beyond theming/overlays is required and is documented only
  in external ComplexHeatmap/ggplot2 docs rather than this package
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MultiModalGraphics
  role: Core package providing the ThresholdedScatterplot, ThresholdedScatterplot_table,
    and ThresholdedScatterplot_MAE S4 classes and createPlot() method for thresholded
    faceted volcano plots
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: MultiAssayExperiment
  role: Bioconductor container accepted as the MAE input path (ThresholdedScatterplot_MAE)
    for multi-omics volcano plot generation
- name: ggplot2
  role: Plotting engine whose objects are built by createPlot() and whose themes/legend
    parameters customize the faceted volcano panels
- name: limma
  role: Differential-expression engine run under the hood by ThresholdedScatterplot_table
    when constructing volcano data from an expression matrix + metadata
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:706496c977dcdba9040d5d1082012620b2018c8e420b5c1d209a2c33abef5468
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
  - ggplot2
  - knitr
  - limma
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# volcano-plot-threshold-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Embeds fold-change and p-value thresholds into faceted volcano plots via the MultiModalGraphics ThresholdedScatterplot family, automatically classifying features as upregulated, downregulated, or neutral and annotating the per-panel feature counts directly on the figure.

## When to use

Use when rendering faceted volcano plots (log2 fold-change vs. -log10 p-value) from DE tables, expression matrices + metadata, or MultiAssayExperiment objects and you need per-facet up/down/neutral counts annotated at explicit thresholds.

## When NOT to use

- The goal is an annotated heatmap with significance markers rather than a volcano plot — use AnnotatedHeatmap or CompositeFeatureHeatmap from the same package instead
- Only raw expression data without group/sample metadata is available and no differential-expression pipeline can be run, since the volcano requires per-feature fold-change and p-value values
- Advanced customization beyond theming/overlays is required and is documented only in external ComplexHeatmap/ggplot2 docs rather than this package

## Inputs

- Precomputed DE data.frame with log2fc, negLog10p, regulation, SampleType, and optional timePoint columns
- Expression or count matrix (expr) plus sample metadata (meta) with a groupColumn, e.g. 100 genes x 20 samples
- MultiAssayExperiment object (e.g. miniACC) with an assayName, groupColumn, sampleType, and timepoint column
- Threshold parameters: highLog2fc, lowLog2fc, negLog10pValue (or pvalue_cutoff, fc_cutoff)

## Outputs

- Faceted ggplot2 volcano plot (log2 fold-change vs. -log10 p-value) with per-panel up/down/neutral feature-count annotations
- Rendered figure file (e.g. thresholded_scatterplot_mae.png via png()/pdf())
- Extracted per-panel threshold counts (e.g. threshold_counts.csv)

## How to apply

Install MultiModalGraphics from github.com/famanalytics0/MultiModalGraphics together with MultiAssayExperiment and ggplot2, then choose the constructor matching the input: ThresholdedScatterplot for precomputed DE tables (with columns log2fc, negLog10p, regulation, SampleType, timePoint), ThresholdedScatterplot_table for expression/count matrices + metadata (limma/voom runs per SampleType-timePoint cell after variance filtering via var_quantile=0.75), or ThresholdedScatterplot_MAE for MultiAssayExperiment objects (e.g. miniACC, assayName='RNASeq2GeneNorm'). Set the cutoffs explicitly: highLog2fc=1.0 classifies log2FC > 1 as 'up', lowLog2fc=-1.0 classifies log2FC < -1 as 'down', and negLog10pValue=1.301 corresponds to p < 0.05. Call createPlot() with color1/color2/color3 for down/neutral/up points and optional text_size, point_size, legend_labels, and custom_theme arguments, then show() or export via pdf()/png(). The result is a faceted volcano plot in which each panel carries counts of the thresholded upregulated and downregulated features; verify by recomputing counts from the data against the annotated panel numbers.

## Related tools

- **MultiModalGraphics** (Core package providing the ThresholdedScatterplot, ThresholdedScatterplot_table, and ThresholdedScatterplot_MAE S4 classes and createPlot() method for thresholded faceted volcano plots) — https://github.com/famanalytics0/MultiModalGraphics
- **MultiAssayExperiment** (Bioconductor container accepted as the MAE input path (ThresholdedScatterplot_MAE) for multi-omics volcano plot generation)
- **ggplot2** (Plotting engine whose objects are built by createPlot() and whose themes/legend parameters customize the faceted volcano panels)
- **limma** (Differential-expression engine run under the hood by ThresholdedScatterplot_table when constructing volcano data from an expression matrix + metadata)

## Examples

```
cs1 <- ThresholdedScatterplot(data = df_de, highLog2fc = 1.0, lowLog2fc = -1.0, negLog10pValue = 1.301); cs1 <- createPlot(cs1, color1 = "tomato", color2 = "grey50", color3 = "steelblue"); show(cs1)
```

## Evaluation signals

- Each rendered facet displays counts of upregulated and downregulated features that match counts recomputed from the DE table at the supplied highLog2fc/lowLog2fc/negLog10pValue cutoffs
- Recomputed counts written to threshold_counts.csv agree with the counts annotated in the vignette's rendered figures
- Points in each panel are colored consistently with the legend mapping (e.g. tomato='down', grey='neutral', steelblue='up') and split by SampleType/timePoint facets
- The same annotated faceted panels are produced whether the input was a precomputed DE table or a MultiAssayExperiment run through ThresholdedScatterplot_MAE
- Cutoffs chosen by the user are honored: with highLog2fc=1.0, lowLog2fc=-1.0, and negLog10pValue=1.301, only features with |log2FC| > 1 and -log10(p) > 1.301 (p < 0.05) are labeled up or down

## Limitations

- The vignette claims to include examples (data generation, scatterplots, informative heatmaps, multifeature grids) but no such example content appears in the document body; advanced customization is deferred to external ComplexHeatmap/ggplot2 documentation
- Package and dependency version numbers are not reported in the article text (sessionInfo() appears only as unevaluated code), though the README lists version 0.99.5
- No named datasets or accession identifiers are provided in the vignette for the biological/multi-omics data demonstrated

## Evidence

- [readme] README threshold parameters and per-facet count annotation: [evidence span withheld — anchor status: external_doc]
- [readme] README result description of thresholded volcano with counts: [evidence span withheld — anchor status: external_doc]
- [readme] README variance filtering parameter for matrix input path: [evidence span withheld — anchor status: external_doc]
