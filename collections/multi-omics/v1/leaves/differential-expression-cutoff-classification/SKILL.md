---
name: differential-expression-cutoff-classification
description: Use when a task requires thresholding differential-expression results
  into up/down/neutral classes and annotating per-panel counts on volcano/scatterplot
  facets from DE tables, matrices+metadata, or MultiAssayExperiment objects.
when_to_use_negative:
- Input is a fold-change/p-value matrix intended for annotated heatmaps rather than
  volcano panels — use AnnotatedHeatmap/CompositeFeatureHeatmap instead
- Only a single unstratified panel is needed with no faceting by SampleType or timePoint
  and no per-panel count annotations
- No fold-change or p-value information exists and the goal is unsupervised exploration
  — use MultiModalPlot() for rapid multimodal figure generation instead
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: MultiModalGraphics
  role: R package supplying the ThresholdedScatterplot, ThresholdedScatterplot_table,
    and ThresholdedScatterplot_MAE classes that apply cutoffs and annotate per-panel
    up/down/neutral counts
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: MultiAssayExperiment
  role: Bioconductor container for the MAE input path (ThresholdedScatterplot_MAE)
    exercised on synthetic MAE objects and miniACC
- name: limma
  role: Differential expression engine run under the hood by ThresholdedScatterplot_table
    (voom+limma per SampleType-timePoint cell)
- name: ggplot2
  role: Rendering engine for the faceted volcano plot produced by createPlot
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:b93fc03fc14c0918c4c39182240ec347961ed764e7f8491228695e177c421bdd
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
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# differential-expression-cutoff-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify features (genes/proteins) into upregulated, downregulated, and neutral classes on faceted volcano panels by applying user-supplied log2 fold-change and -log10 p-value thresholds, and annotate per-panel feature counts, as implemented in MultiModalGraphics' ThresholdedScatterplot classes.

## When to use

Use when a task requires thresholding differential-expression results into up/down/neutral classes and annotating per-panel counts on volcano/scatterplot facets from DE tables, matrices+metadata, or MultiAssayExperiment objects.

## When NOT to use

- Input is a fold-change/p-value matrix intended for annotated heatmaps rather than volcano panels — use AnnotatedHeatmap/CompositeFeatureHeatmap instead
- Only a single unstratified panel is needed with no faceting by SampleType or timePoint and no per-panel count annotations
- No fold-change or p-value information exists and the goal is unsupervised exploration — use MultiModalPlot() for rapid multimodal figure generation instead

## Inputs

- Precomputed DE table (data.frame with log2fc, negLog10p, regulation, SampleType, optional timePoint columns)
- Expression/counts matrix (e.g. matrix of rpois counts, genes x samples) plus sample metadata data.frame (groupColumn, sampleType, timePoint)
- MultiAssayExperiment object (e.g. miniACC) with a named assay (e.g. RNASeq2GeneNorm) and colData columns for groupColumn/sampleType/timepoint

## Outputs

- Faceted volcano plot (log2 fold-change vs. -log10 p-value) with up/down/neutral points colored by threshold class
- Per-panel annotated counts of upregulated, downregulated, and neutral features
- Exportable count table (e.g. threshold_counts.csv) and figure file (e.g. thresholded_scatterplot_mae.png)

## How to apply

Construct a ThresholdedScatterplot object from a precomputed DE table with columns log2fc, negLog10p, regulation, SampleType, and optional timePoint, supplying highLog2fc (e.g. 1.0), lowLog2fc (e.g. -1.0), and negLog10pValue (e.g. 1.301, equivalent to p < 0.05); for raw/normalized matrices run ThresholdedScatterplot_table, which variance-filters (var_quantile=0.75) and runs voom+limma per SampleType-timePoint cell; for MultiAssayExperiment inputs call ThresholdedScatterplot_MAE with assayName, groupColumn, sampleType, and timepoint columns. Then call createPlot with color1/color2/color3 for down/neutral/up classes to render faceted volcano panels (log2 fold-change vs. -log10 p-value), where the class automatically annotates within each panel the number of upregulated, downregulated, and neutral features. Extract those annotated counts (e.g. to threshold_counts.csv) and compare against independent computation to verify correctness.

## Related tools

- **MultiModalGraphics** (R package supplying the ThresholdedScatterplot, ThresholdedScatterplot_table, and ThresholdedScatterplot_MAE classes that apply cutoffs and annotate per-panel up/down/neutral counts) — https://github.com/famanalytics0/MultiModalGraphics
- **MultiAssayExperiment** (Bioconductor container for the MAE input path (ThresholdedScatterplot_MAE) exercised on synthetic MAE objects and miniACC)
- **limma** (Differential expression engine run under the hood by ThresholdedScatterplot_table (voom+limma per SampleType-timePoint cell))
- **ggplot2** (Rendering engine for the faceted volcano plot produced by createPlot)

## Examples

```
cs1 <- ThresholdedScatterplot(df_de, highLog2fc=1.0, lowLog2fc=-1.0, negLog10pValue=1.301); cs1 <- createPlot(cs1, color1="tomato", color2="grey50", color3="steelblue"); show(cs1)
```

## Evaluation signals

- Each rendered facet displays annotated counts of upregulated, downregulated, and neutral features matching counts independently recomputed from the cutoffs (log2fc > highLog2fc, log2fc < lowLog2fc, -log10p > negLog10pValue)
- The -log10 p-value cutoff maps correctly to a p-value threshold (e.g. negLog10pValue = 1.301 implies p < 0.05)
- Extracted per-panel counts written to threshold_counts.csv match the counts annotated in the vignette's rendered figures
- The MAE path (ThresholdedScatterplot_MAE) produces the same faceted volcano panels and annotations as the precomputed-table path on equivalent DE inputs
- Points are colored per the color1/color2/color3 arguments (down/neutral/up) and plot honors xlab, ylab, legend_position, and custom_theme overrides

## Limitations

- Per the README, the default color mapping in examples is inverted/confusing (color1='tomato' is labeled 'up = steelblue' in a comment), so users must verify the color-to-class mapping when interpreting panels
- Matrix+metadata and MAE paths depend on variance filtering (var_quantile=0.75) and voom+limma per stratum, so counts depend on the filtering and parallel (BPPARAM) settings
- The article vignette defers advanced customization to external ComplexHeatmap/ggplot2 documentation and does not report package/dependency version numbers in evaluated sessionInfo output

## Evidence

- [other] The ThresholdedScatterplot class automatically applies fold-change and p-value cutoffs and annotates, within each panel, the number of upregulated, downregulated, and neutral features.: [evidence span withheld — anchor status: not_found]
- [readme] README annotation result: [evidence span withheld — anchor status: external_doc]
- [readme] README table-path pipeline: [evidence span withheld — anchor status: external_doc]
- [readme] README customization parameters: [evidence span withheld — anchor status: external_doc]
