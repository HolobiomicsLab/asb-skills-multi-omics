---
name: s4-class-dispatch-implementation
description: Use when an R analysis must accept heterogeneous multi-omics inputs (raw
  expression matrix, precomputed DE table, or MultiAssayExperiment object) and one
  unified plotting entry point must detect the type and dispatch to the right visualization
  class.
when_to_use_negative:
- The input is already a specialized shape matched to a single class and only one
  plot type is needed - call the specific S4 class directly (e.g. ThresholdedScatterplot
  for a precomputed DE table) instead of the dispatch wrapper
- The input is a pathway/feature grid table for 2D multifeature grid heatmaps, or
  a list of multi-cohort/multi-timepoint objects, which fall outside the three dispatch
  types the wrapper detects
- The working environment is not R with Bioconductor (MultiAssayExperiment, ComplexHeatmap,
  ggplot2 available) - the dispatch pattern relies on S4 inheritance testing in R
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MultiModalGraphics
  role: Provides the MultiModalPlot() wrapper and the target S4 classes (ThresholdedScatterplot,
    AnnotatedHeatmap, CompositeFeatureHeatmap) that the dispatch routes to
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: MultiAssayExperiment
  role: Supplies the MAE container class used in the is(object, 'MultiAssayExperiment')
    inheritance test that triggers MAE dispatch
- name: ggplot2
  role: Produces the returned ggplot objects for the faceted volcano plots built by
    the dispatched classes via createPlot()
- name: ComplexHeatmap
  role: Underlying heatmap engine whose objects can be extracted from the dispatched
    heatmap classes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:e7330c8ab41f0245884c294a8ae17298580be366b06d6add2767add4ed595657
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
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# s4-class-dispatch-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements the MultiModalPlot() wrapper's automatic input-type detection and dispatch pattern from the MultiModalGraphics R/Bioconductor package, routing raw matrices, precomputed DE tables, and MultiAssayExperiment objects to the appropriate S4 visualization class (ThresholdedScatterplot, AnnotatedHeatmap, or CompositeFeatureHeatmap) and returning the underlying S4 or ggplot objects.

## When to use

Use when an R analysis must accept heterogeneous multi-omics inputs (raw expression matrix, precomputed DE table, or MultiAssayExperiment object) and one unified plotting entry point must detect the type and dispatch to the right visualization class.

## When NOT to use

- The input is already a specialized shape matched to a single class and only one plot type is needed - call the specific S4 class directly (e.g. ThresholdedScatterplot for a precomputed DE table) instead of the dispatch wrapper
- The input is a pathway/feature grid table for 2D multifeature grid heatmaps, or a list of multi-cohort/multi-timepoint objects, which fall outside the three dispatch types the wrapper detects
- The working environment is not R with Bioconductor (MultiAssayExperiment, ComplexHeatmap, ggplot2 available) - the dispatch pattern relies on S4 inheritance testing in R

## Inputs

- Raw expression/count matrix (e.g. genes x samples matrix with rownames/colnames)
- Precomputed differential expression table with columns log2fc, negLog10p, regulation, SampleType, [optional timePoint]
- MultiAssayExperiment (MAE) object, e.g. miniACC from MultiAssayExperiment
- Expression matrix + sample metadata (expr, meta) handled via downstream classes like ThresholdedScatterplot_table

## Outputs

- Dispatched S4 visualization object (ThresholdedScatterplot, AnnotatedHeatmap, or CompositeFeatureHeatmap)
- Underlying ggplot object from createPlot() (faceted volcano plot: log2 fold-change vs. -log10(p-value))
- Per-input routing outcome record (which class was dispatched; S4 vs ggplot return type), e.g. written to a CSV such as multimodalplot_routing_results.csv

## How to apply

Implement a single wrapper function (MultiModalPlot()) that inspects the input with is.matrix()/is.data.frame() checks and MultiAssayExperiment::is(object, 'MultiAssayExperiment') inheritance testing, then routes each detected type to its target S4 class: raw matrix to ThresholdedScatterplot, precomputed statistical table (log2fc, negLog10p, regulation, SampleType, optional timePoint columns) to AnnotatedHeatmap, and MAE object to CompositeFeatureHeatmap. The wrapper must return the underlying S4 or ggplot object rather than only rendering a side-effect plot, preserving transparency and extensibility per the package's dual design where MultiModalPlot() is the convenient entry point and the core S4 classes remain the primary interface for advanced customization. Exercise the wrapper on each example input shape (e.g. the package extdata files from famanalytics0/MultiModalGraphics) and record which class was dispatched and whether the returned object is an S4 or ggplot instance. Threshold parameters such as highLog2fc = 1.0, lowLog2fc = -1.0, and negLog10pValue = 1.301 (p < 0.05) are then applied inside the dispatched class via createPlot().

## Related tools

- **MultiModalGraphics** (Provides the MultiModalPlot() wrapper and the target S4 classes (ThresholdedScatterplot, AnnotatedHeatmap, CompositeFeatureHeatmap) that the dispatch routes to) — https://github.com/famanalytics0/MultiModalGraphics
- **MultiAssayExperiment** (Supplies the MAE container class used in the is(object, 'MultiAssayExperiment') inheritance test that triggers MAE dispatch)
- **ggplot2** (Produces the returned ggplot objects for the faceted volcano plots built by the dispatched classes via createPlot())
- **ComplexHeatmap** (Underlying heatmap engine whose objects can be extracted from the dispatched heatmap classes)

## Examples

```
cs1 <- ThresholdedScatterplot(data = df_de, highLog2fc = 1.0, lowLog2fc = -1.0, negLog10pValue = 1.301); cs1 <- createPlot(cs1, color1 = "tomato", color2 = "grey50", color3 = "steelblue"); show(cs1)
```

## Evaluation signals

- For each test input, the recorded dispatch outcome matches the expected routing: raw matrix -> ThresholdedScatterplot, precomputed DE table -> AnnotatedHeatmap, MAE object -> CompositeFeatureHeatmap
- The wrapper's return value is an S4 or ggplot instance, not NULL or a side-effect-only plot (verify with class() / is(object))
- show(createPlot(...)) on the returned object renders the expected faceted volcano plot (log2fc vs -log10p) with up/down/neutral counts annotated per facet
- MAE inputs are accepted without manual conversion (inheritance test succeeds), and non-MAE inputs do not falsely route to CompositeFeatureHeatmap
- Threshold parameters (highLog2fc, lowLog2fc, negLog10pValue) propagate through the dispatched class rather than being hard-coded

## Limitations

- The vignette claims to include examples (data generation, scatterplots, informative heatmaps, multifeature grids) but no such example content appears in the document body, so dispatch behavior must be verified against the GitHub repository's extdata examples
- Package and dependency version numbers are not reported; session info appears only as unevaluated code, complicating reproducibility checks
- Advanced features and customization are not documented in the guide; readers are deferred to external ComplexHeatmap and ggplot2 documentation
- No changelog artifact was found, and no named datasets or accession identifiers are provided for the demonstrated multi-omics data

## Evidence

- [readme] README on extracting returned plot objects: [evidence span withheld — anchor status: external_doc]
