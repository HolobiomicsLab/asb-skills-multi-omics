---
name: multiomics-object-routing-to-visualization-classes
description: Use when you hold multi-omics input in mixed shapes — a raw expression/count
  matrix, a precomputed DE table (log2fc, negLog10p), or a MultiAssayExperiment object
  — and want one call to auto-route it to the correct volcano/heatmap class without
  manually choosing a plotting class.
when_to_use_negative:
- Input is already a specialized object requiring a specific class constructor — call
  ThresholdedScatterplot(), ThresholdedScatterplot_MAE(), etc. directly instead of
  relying on dispatch
- You need advanced customization or method development on the S4 classes themselves
  — the article states 'the core S4 classes remain the primary interface for advanced
  customization and method development'
- Input is a pathway/feature grid table for 2D/3D grid heatmaps or a list of multi-cohort
  objects, which the wrapper's three-way dispatch does not cover
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: MultiModalGraphics
  role: Source package providing MultiModalPlot(), the S4 classes (ThresholdedScatterplot,
    AnnotatedHeatmap, CompositeFeatureHeatmap), and createPlot()/show() methods being
    dispatched to
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: MultiAssayExperiment
  role: Bioconductor container whose objects are detected via inheritance testing
    and routed to the MAE-aware visualization classes
- name: ggplot2
  role: Rendering backend for the faceted volcano plots returned by createPlot() on
    dispatched objects
- name: limma
  role: Differential expression engine invoked under the hood when routing matrix+metadata
    inputs to visualization classes
- name: ComplexHeatmap
  role: Heatmap rendering engine for dispatched heatmap classes (AnnotatedHeatmap,
    CompositeFeatureHeatmap)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:bf5bbab4b10bc4eb536d26f271772b2858aa88c95c5d20d3303664471a2bdea3
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
  - limma
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# multiomics-object-routing-to-visualization-classes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements the MultiModalPlot() high-level wrapper from the MultiModalGraphics R package, which detects whether the input is a raw matrix, precomputed DE statistical table, or MultiAssayExperiment object, and dispatches to the matching S4 visualization class (ThresholdedScatterplot, AnnotatedHeatmap, CompositeFeatureHeatmap) while returning the underlying S4/ggplot objects for transparency and extensibility.

## When to use

Use when you hold multi-omics input in mixed shapes — a raw expression/count matrix, a precomputed DE table (log2fc, negLog10p), or a MultiAssayExperiment object — and want one call to auto-route it to the correct volcano/heatmap class without manually choosing a plotting class.

## When NOT to use

- Input is already a specialized object requiring a specific class constructor — call ThresholdedScatterplot(), ThresholdedScatterplot_MAE(), etc. directly instead of relying on dispatch
- You need advanced customization or method development on the S4 classes themselves — the article states 'the core S4 classes remain the primary interface for advanced customization and method development'
- Input is a pathway/feature grid table for 2D/3D grid heatmaps or a list of multi-cohort objects, which the wrapper's three-way dispatch does not cover

## Inputs

- Raw expression/count matrix (e.g., 100 genes x 20 samples matrix with rownames/colnames)
- Precomputed DE statistics data.frame with columns log2fc, negLog10p, regulation, SampleType, optional timePoint
- MultiAssayExperiment (MAE) object (e.g., miniACC from MultiAssayExperiment)
- Expression matrix plus sample metadata data.frame (expr, meta) for limma/voom DE pipelines

## Outputs

- S4 visualization object of the dispatched class (ThresholdedScatterplot, AnnotatedHeatmap, or CompositeFeatureHeatmap)
- ggplot instance (volcano plot) or ComplexHeatmap object after createPlot()/show()
- Faceted volcano plot annotated with up/down/neutral counts per facet
- Per-input routing outcomes record (e.g., multimodalplot_routing_results.csv)

## How to apply

Detect the input type programmatically: use is.matrix()/is.data.frame() checks plus MultiAssayExperiment::is(object, 'MultiAssayExperiment') inheritance testing. Route raw matrices to ThresholdedScatterplot, precomputed statistical tables (log2fc, negLog10p, regulation, SampleType, optional timePoint columns) to AnnotatedHeatmap, and MAE objects to CompositeFeatureHeatmap. After dispatch, call createPlot() on the returned S4 object to build the ggplot/ComplexHeatmap instance, and return the underlying S4 or ggplot object rather than only drawing as a side effect, so expert users can customize (e.g., thresholds highLog2fc, lowLog2fc, negLog10pValue) and extract plots for RMarkdown. Rationale: the dual design keeps one-line entry for novices while the core S4 classes remain the primary interface for advanced customization and method development.

## Related tools

- **MultiModalGraphics** (Source package providing MultiModalPlot(), the S4 classes (ThresholdedScatterplot, AnnotatedHeatmap, CompositeFeatureHeatmap), and createPlot()/show() methods being dispatched to) — https://github.com/famanalytics0/MultiModalGraphics
- **MultiAssayExperiment** (Bioconductor container whose objects are detected via inheritance testing and routed to the MAE-aware visualization classes)
- **ggplot2** (Rendering backend for the faceted volcano plots returned by createPlot() on dispatched objects)
- **limma** (Differential expression engine invoked under the hood when routing matrix+metadata inputs to visualization classes)
- **ComplexHeatmap** (Heatmap rendering engine for dispatched heatmap classes (AnnotatedHeatmap, CompositeFeatureHeatmap))

## Examples

```
cs <- ThresholdedScatterplot(df); show(createPlot(cs, color1="tomato", color2="lightgrey", color3="steelblue", title="Volcano Plot"))
```

## Evaluation signals

- Each tested input shape dispatches to the expected class: raw matrix -> ThresholdedScatterplot, DE table -> AnnotatedHeatmap, MAE -> CompositeFeatureHeatmap
- Returned object is a valid S4 or ggplot instance (not just a side-effect plot); class(x) checks pass
- createPlot() on the dispatched object renders a faceted volcano with up/down/neutral points colored and per-facet count annotations
- MAE route runs on miniACC (MultiAssayExperiment) with MulticoreParam(workers=2) without erroring on assayName/groupColumn arguments
- Routing outcomes for all extdata test inputs are recorded (e.g., multimodalplot_routing_results.csv) and match the expected per-input class

## Limitations

- The README documents per-class constructors (ThresholdedScatterplot, AnnotatedHeatmap, CompositeFeatureHeatmap) and MAE variants, but the vignette body lacks worked example content for the unified wrapper, so dispatch behavior must be verified empirically against extdata files
- Advanced features and customization are not documented in the guide; users are deferred to external ComplexHeatmap and ggplot2 documentation
- Package and dependency version numbers are not reported in the document text (sessionInfo() appears only as unevaluated code), so dispatch behavior may vary across versions (README reports Version 0.99.5)
- No named datasets or accession identifiers are provided in the vignette for the biological data demonstrated

## Evidence

- [readme] README defines the input shapes the wrapper must discriminate: [evidence span withheld — anchor status: external_doc]
