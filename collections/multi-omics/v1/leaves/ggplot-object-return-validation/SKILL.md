---
name: ggplot-object-return-validation
description: Use when wrapping MultiModalGraphics visualization calls (raw matrix,
  precomputed DE table, or MultiAssayExperiment input) and you need the returned S4/ggplot
  object for further annotation, theming, or RMarkdown embedding instead of a rendered
  figure.
when_to_use_negative:
- The goal is only a quick one-off rendered figure with no downstream programmatic
  access, theming, or RMarkdown annotation
- Input is a pathway/feature grid table for 2D/3D multifeature grids or a Shiny interactive
  dashboard — different package entry points apply
- Advanced customization is needed at the DE-pipeline level (variance filtering, voom+limma
  per SampleType-timePoint cell); use the class-specific *_table or *_MAE constructors
  directly
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MultiModalGraphics
  role: Provides the S4 classes (ThresholdedScatterplot, AnnotatedHeatmap, CompositeFeatureHeatmap)
    and the MultiModalPlot() wrapper whose return values are validated
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: ggplot2
  role: Rendering engine whose ggplot object should be returned by createPlot() for
    RMarkdown and annotation reuse
- name: ComplexHeatmap
  role: Heatmap backend; heatmap-producing classes should return ComplexHeatmap objects
    extractable for reporting
- name: MultiAssayExperiment
  role: MAE input container detected by inheritance testing to route toward MAE-specific
    visualization classes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:291e74db1ca42c14fd76b31202ae983ce2df2c3127655b8e6cd6a5ac7188d223
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
  - MultiAssayExperiment
  - MultiModalGraphics
  - R
  - ggplot2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# ggplot-object-return-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validate that MultiModalGraphics plotting wrappers (e.g., MultiModalPlot(), ThresholdedScatterplot + createPlot) return the underlying S4 or ggplot/ComplexHeatmap objects rather than producing only side-effect plots, preserving transparency and extensibility for downstream customization.

## When to use

Use when wrapping MultiModalGraphics visualization calls (raw matrix, precomputed DE table, or MultiAssayExperiment input) and you need the returned S4/ggplot object for further annotation, theming, or RMarkdown embedding instead of a rendered figure.

## When NOT to use

- The goal is only a quick one-off rendered figure with no downstream programmatic access, theming, or RMarkdown annotation
- Input is a pathway/feature grid table for 2D/3D multifeature grids or a Shiny interactive dashboard — different package entry points apply
- Advanced customization is needed at the DE-pipeline level (variance filtering, voom+limma per SampleType-timePoint cell); use the class-specific *_table or *_MAE constructors directly

## Inputs

- Raw expression/count matrix (e.g., genes x samples matrix with row/col names) plus sample metadata data.frame
- Precomputed DE table data.frame with columns log2fc, negLog10p, regulation, SampleType, [timePoint]
- MultiAssayExperiment (MAE) object, e.g., miniACC with assayName 'RNASeq2GeneNorm'

## Outputs

- S4 object of class ThresholdedScatterplot, AnnotatedHeatmap, or CompositeFeatureHeatmap
- ggplot object (volcano plot) or ComplexHeatmap object obtained via createPlot()
- Rendered faceted volcano plot with up/down/neutral coloring and per-facet up/down counts when passed to show()

## How to apply

First detect the input shape: a raw expression/count matrix, a precomputed differential-expression table (columns log2fc, negLog10p, regulation, SampleType, optional timePoint), or a MultiAssayExperiment object; dispatch accordingly (matrix/table routes toward ThresholdedScatterplot, MAE routes toward ThresholdedScatterplot_MAE or CompositeFeatureHeatmap). Construct the S4 object (e.g., ThresholdedScatterplot with highLog2fc = 1.0, lowLog2fc = -1.0, negLog10pValue = 1.301 corresponding to p < 0.05) and then call createPlot() to attach the ggplot. Critically, capture the assignment (cs <- ThresholdedScatterplot(df); cs <- createPlot(cs)) instead of relying on device side effects, then verify the returned object's class — S4 ThresholdedScatterplot/AnnotatedHeatmap/CompositeFeatureHeatmap or a ggplot instance — before rendering with show(). This mirrors the package's dual design: the wrapper offers a convenient entry point while the core S4 classes remain the primary interface for advanced customization.

## Related tools

- **MultiModalGraphics** (Provides the S4 classes (ThresholdedScatterplot, AnnotatedHeatmap, CompositeFeatureHeatmap) and the MultiModalPlot() wrapper whose return values are validated) — https://github.com/famanalytics0/MultiModalGraphics
- **ggplot2** (Rendering engine whose ggplot object should be returned by createPlot() for RMarkdown and annotation reuse)
- **ComplexHeatmap** (Heatmap backend; heatmap-producing classes should return ComplexHeatmap objects extractable for reporting)
- **MultiAssayExperiment** (MAE input container detected by inheritance testing to route toward MAE-specific visualization classes)

## Examples

```
cs <- ThresholdedScatterplot(df, highLog2fc = 1.0, lowLog2fc = -1.0, negLog10pValue = 1.301); cs <- createPlot(cs); class(cs); show(cs)
```

## Evaluation signals

- The assigned variable is non-NULL and inherits the expected class (e.g., is(cs, 'ThresholdedScatterplot') is TRUE; createPlot(cs) returns a ggplot instance), not a side-effect-only render
- Dispatch matches input type: a data.frame DE table, an expression matrix + meta, and a MultiAssayExperiment each route to the expected S4 class and record the dispatched class per input
- show(cs) renders a faceted volcano plot with up/down/neutral colors and per-facet up/down count annotations as documented
- Threshold parameters are respected: points flagged up only when log2FC > 1.0 and -log10(p) > 1.301 (p < 0.05)
- Returned ggplot/ComplexHeatmap object can be embedded in RMarkdown or wrapped in pdf()/png() export, confirming the object (not just the device output) was preserved

## Limitations

- The vignette defers advanced customization and theming to external ComplexHeatmap and ggplot2 documentation rather than documenting it internally
- The vignette claims example content (data generation, scatterplots, informative heatmaps, multifeature grids) but no such example content appears in the document body; sessionInfo() appears only as unevaluated code with no dependency versions
- Assignment order matters: the object must be created before createPlot/show, otherwise 'object cs not found' errors occur

## Evidence

- [readme] Extract objects for RMarkdown: [evidence span withheld — anchor status: not_found]
