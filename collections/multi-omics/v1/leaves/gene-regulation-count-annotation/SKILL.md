---
name: gene-regulation-count-annotation
description: Use when you have a precomputed DE table (log2fc, negLog10p, SampleType,
  timePoint) or matrix+metadata and need faceted volcano plots with per-panel up/down/neutral
  gene counts annotated.
when_to_use_negative:
- The goal is an informative heatmap with overlaid significance markers (AnnotatedHeatmap/CompositeFeatureHeatmap)
  rather than a volcano-style scatterplot
- The input is already a per-facet regulation count table and no visualization or
  threshold reclassification is needed
- The dataset lacks SampleType/timePoint-style grouping variables so faceted per-panel
  count annotation is not meaningful
edam_operation: http://edamontology.org/operation_3200
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: MultiModalGraphics
  role: Provides the ThresholdedScatterplot S4 class and createPlot() method that
    apply fold-change/p-value cutoffs and annotate per-facet regulation counts
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: ggplot2
  role: Rendering engine for the faceted volcano scatterplot; used for theming and
    customization
- name: dplyr
  role: Data wrangling to compute log2 fold-change and -log10 p-value plotting variables
    from the DE table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:07821553d78da49ff2f622ec450a6711ee13360fc8b3adfa88e3949c9b91d658
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
  edam_operation: http://edamontology.org/operation_3200
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# gene-regulation-count-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A MultiModalGraphics ThresholdedScatterplot workflow that classifies features as up-, down-, or non-significant in faceted volcano panels and annotates each panel with regulation counts. It provides automatic quantitative summaries (fold-change/p-value cutoffs) per facet, as demonstrated on the GSE68077-derived differential-expression example.

## When to use

Use when you have a precomputed DE table (log2fc, negLog10p, SampleType, timePoint) or matrix+metadata and need faceted volcano plots with per-panel up/down/neutral gene counts annotated.

## When NOT to use

- The goal is an informative heatmap with overlaid significance markers (AnnotatedHeatmap/CompositeFeatureHeatmap) rather than a volcano-style scatterplot
- The input is already a per-facet regulation count table and no visualization or threshold reclassification is needed
- The dataset lacks SampleType/timePoint-style grouping variables so faceted per-panel count annotation is not meaningful

## Inputs

- Precomputed differential-expression table with columns log2fc, negLog10p, regulation, SampleType, timePoint (e.g., extdata/scatterexample.csv or GSE68077-derived table)
- Expression/count matrix plus sample metadata (expr, meta) handled via ThresholdedScatterplot_table with limma/voom
- MultiAssayExperiment object handled via ThresholdedScatterplot_MAE

## Outputs

- Faceted volcano scatterplot (log2FC vs -log10 p) saved as e.g. thresholded_scatterplot.png with up/down/neutral points colored and per-facet regulation counts annotated
- Summary table of per-panel up-regulated and down-regulated gene counts

## How to apply

Load the DE table (e.g., extdata/scatterexample.csv from the MultiModalGraphics GitHub repo, or a GEO GSE68077-derived table) into R with dplyr. Compute plotting variables: log2 fold change and -log10(p-value) from fold-change and p-value columns. Apply significance thresholds — |log2fc| > 0.585 (~1.5-fold change, set via highLog2fc/lowLog2fc) and -log10p > 1.301 (p < 0.05, set via negLog10pValue) — to label genes as up, down, or neutral. Construct a ThresholdedScatterplot S4 object, then call createPlot() with color1/color2/color3 for down/neutral/up points, faceting by SampleType (X) and timePoint (Y); the class automatically annotates counts of up- and down-regulated features in each facet. Export the multi-panel figure via png()/pdf() and extract the per-panel up/down counts into a summary table.

## Related tools

- **MultiModalGraphics** (Provides the ThresholdedScatterplot S4 class and createPlot() method that apply fold-change/p-value cutoffs and annotate per-facet regulation counts) — https://github.com/famanalytics0/MultiModalGraphics
- **ggplot2** (Rendering engine for the faceted volcano scatterplot; used for theming and customization)
- **dplyr** (Data wrangling to compute log2 fold-change and -log10 p-value plotting variables from the DE table)

## Examples

```
cs1 <- ThresholdedScatterplot(data = df_de, highLog2fc = 1.0, lowLog2fc = -1.0, negLog10pValue = 1.301); cs1 <- createPlot(cs1, color1 = "tomato", color2 = "grey50", color3 = "steelblue", title = "Example Volcano"); show(cs1)
```

## Evaluation signals

- Each facet displays an annotation of the counts of up- and down-regulated features, and these counts match a manual tally using the same thresholds (e.g., |log2fc| > 0.585, -log10p > 1.301)
- Points are partitioned into exactly three regulation classes (up, down, neutral) colored per the color1/color2/color3 arguments, with no unclassified points
- The plot contains one panel per SampleType-by-timePoint combination present in the data (e.g., T5R1, T5R10, T10R1, T10R28, T10R42)
- X-axis spans positive and negative log2 fold change with vertical cutoff lines at ±highLog2fc/lowLog2fc; Y-axis is -log10(p-value) with the 1.301 cutoff visible
- The figure exports cleanly via png()/pdf() and the returned ggplot object retains the annotated counts in its facet labels

## Limitations

- Package/dependency versions are not reported in the vignette (sessionInfo() appears only as unevaluated code), so reproducibility of exact rendering may vary
- Advanced customization is deferred to external ComplexHeatmap and ggplot2 documentation rather than being documented in the guide itself
- No named datasets or accession identifiers (beyond the example CSV) are provided in the vignette; users must map their own DE table schema onto log2fc/negLog10p/regulation/SampleType/timePoint columns

## Evidence

- [readme] README result for smoke-test example: [evidence span withheld — anchor status: not_found]
- [readme] README threshold parameter comment: [evidence span withheld — anchor status: external_doc]
