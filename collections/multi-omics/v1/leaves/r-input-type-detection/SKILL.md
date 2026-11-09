---
name: r-input-type-detection
description: Use when designing or reimplementing a unified R entry-point function
  (like MultiModalPlot()) that must auto-detect heterogeneous input shapes—raw matrices,
  DE tables, MAE objects—and route them to class-specific plotting constructors.
when_to_use_negative:
- Input is already a constructed S4 plotting object — call createPlot()/show() directly
  instead of re-dispatching through the wrapper
- Input is a pathway/feature grid table destined for 2D grid heatmaps, a distinct
  input class outside the matrix/DE-table/MAE dispatch paths
- The goal is advanced customization or method development on core S4 classes — those
  are the primary interface, not the convenience wrapper
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MultiModalGraphics
  role: Package providing the MultiModalPlot() wrapper and the ThresholdedScatterplot
    / AnnotatedHeatmap / CompositeFeatureHeatmap S4 classes that the dispatcher routes
    to
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: MultiAssayExperiment
  role: Bioconductor container whose objects are detected via inheritance testing
    and dispatched to MAE-specific plot classes
- name: ggplot2
  role: Engine behind the ggplot objects returned by createPlot() after dispatch
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:84eaa35c996b7c52e437e3ecb2ca31c7b19bf1a1e7761bd1d3167f99a5b79144
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# r-input-type-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pattern for writing high-level R wrapper functions that detect whether caller-supplied input is a raw matrix/data.frame, a precomputed statistics table, or a Bioconductor container (MultiAssayExperiment), and dispatch to the appropriate S4 visualization class while returning the underlying object for transparency.

## When to use

Use when designing or reimplementing a unified R entry-point function (like MultiModalPlot()) that must auto-detect heterogeneous input shapes—raw matrices, DE tables, MAE objects—and route them to class-specific plotting constructors.

## When NOT to use

- Input is already a constructed S4 plotting object — call createPlot()/show() directly instead of re-dispatching through the wrapper
- Input is a pathway/feature grid table destined for 2D grid heatmaps, a distinct input class outside the matrix/DE-table/MAE dispatch paths
- The goal is advanced customization or method development on core S4 classes — those are the primary interface, not the convenience wrapper

## Inputs

- Raw or normalized expression/count matrix with sample metadata (expr, meta)
- Precomputed differential-expression table (log2fc, negLog10p, regulation, SampleType, optional timePoint)
- MultiAssayExperiment (MAE) object, e.g. miniACC
- Lists of the above (multi-cohort or multi-timepoint)

## Outputs

- Dispatched S4 visualization object (e.g., ThresholdedScatterplot, AnnotatedHeatmap, CompositeFeatureHeatmap)
- ggplot object from createPlot()
- Routing log recording which input type mapped to which class (e.g., multimodalplot_routing_results.csv)

## How to apply

Implement type detection in the wrapper using is.matrix()/is.data.frame() checks plus MultiAssayExperiment::is(object, 'MultiModalGraphics')-style inheritance testing. Route each detected shape to its target S4 class per the package design: precomputed DE tables (log2fc, negLog10p, regulation, SampleType, optional timePoint) go to ThresholdedScatterplot, expression/count matrices with metadata go to ThresholdedScatterplot_table (which runs limma/voom under the hood), and MAE objects go to ThresholdedScatterplot_MAE / CompositeFeatureHeatmap. After dispatch, call createPlot() to build the ggplot, but return the underlying S4 or ggplot objects rather than only side-effect plots, preserving transparency and extensibility. Exercise the wrapper on each input shape (e.g., extdata examples from the famanalytics0/MultiModalGraphics repo) and record which class was dispatched and whether the returned object is an S4 or ggplot instance.

## Related tools

- **MultiModalGraphics** (Package providing the MultiModalPlot() wrapper and the ThresholdedScatterplot / AnnotatedHeatmap / CompositeFeatureHeatmap S4 classes that the dispatcher routes to) — https://github.com/famanalytics0/MultiModalGraphics
- **MultiAssayExperiment** (Bioconductor container whose objects are detected via inheritance testing and dispatched to MAE-specific plot classes)
- **ggplot2** (Engine behind the ggplot objects returned by createPlot() after dispatch)

## Examples

```
cs1 <- ThresholdedScatterplot(data=df_de, highLog2fc=1.0, lowLog2fc=-1.0, negLog10pValue=1.301); cs1 <- createPlot(cs1, color1="tomato", color2="grey50", color3="steelblue"); show(cs1)
```

## Evaluation signals

- Each of the three input shapes (matrix, DE table, MAE) routes to its expected class: matrix -> ThresholdedScatterplot(_table), DE table -> ThresholdedScatterplot, MAE -> ThresholdedScatterplot_MAE / CompositeFeatureHeatmap
- Returned object is an S4 or ggplot instance, not just a side-effect plot, enabling RMarkdown embedding and further customization
- Dispatched objects accept createPlot() parameters (color1/color2/color3, thresholds like highLog2fc=1.0, lowLog2fc=-1.0, negLog10pValue=1.301) without error
- Per-input routing outcomes recorded in multimodalplot_routing_results.csv match the exercised extdata test inputs
- MAE inputs with dataType='auto' correctly trigger automatic count/assay detection downstream (limma/voom DE pipeline)

## Limitations

- The vignette claims example content (data generation, scatterplots, heatmaps, multifeature grids) but no such example body appears; advanced customization is deferred to ComplexHeatmap and ggplot2 documentation
- No named datasets or accession identifiers are provided in the vignette text, and session info/version numbers are only unevaluated code
- The dual wrapper/S4 design means the convenience wrapper is intended for exploratory analysis; core S4 classes remain the primary interface for advanced customization

## Evidence

- [readme] README input-type list: [evidence span withheld — anchor status: not_found]
- [readme] README extensibility: [evidence span withheld — anchor status: external_doc]
