---
name: package-extdata-test-harness-execution
description: Use when validating an R/Bioconductor visualization package against its
  bundled example (extdata) inputs — raw matrices, precomputed DE tables, MAE objects
  — to confirm input-type dispatch and returned object types.
when_to_use_negative:
- The goal is advanced customization or method development — the article states the
  core S4 classes, not the wrapper, are 'the primary interface for advanced customization
  and method development'
- The input is a pathway/feature grid table for 2D multifeature grid heatmaps or list
  inputs for multi-cohort plots, which fall outside the three-shape dispatch being
  tested
- Reproducibility auditing of exact package/dependency versions — the article flags
  that 'Package and dependency version numbers not reported in the document text;
  session info appears only as unevaluated code'
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MultiModalGraphics
  role: Package under test; supplies MultiModalPlot() wrapper, ThresholdedScatterplot/AnnotatedHeatmap/CompositeFeatureHeatmap
    S4 classes, createPlot(), and extdata example files
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: MultiAssayExperiment
  role: Provides the MAE object class (e.g. miniACC) used as one of the three dispatch
    input shapes
- name: ggplot2
  role: ggplot objects returned by createPlot() are the expected output type recorded
    by the harness
- name: limma
  role: Differential expression engine run under the hood when ThresholdedScatterplot_table
    is dispatched on a matrix+metadata input
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:d555483c4788c0d25e98ff988fa1a0b231b738e8e7bf5b12d0a89f55fe328ecb
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
  - limma
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.14285714285714285
n_claims: 7
---

# package-extdata-test-harness-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Executes the MultiModalGraphics package's extdata example datasets as a test harness, running MultiModalPlot()/core classes (ThresholdedScatterplot, AnnotatedHeatmap, CompositeFeatureHeatmap) on each input shape and recording which class is dispatched and what object type is returned.

## When to use

Use when validating an R/Bioconductor visualization package against its bundled example (extdata) inputs — raw matrices, precomputed DE tables, MAE objects — to confirm input-type dispatch and returned object types.

## When NOT to use

- The goal is advanced customization or method development — the article states the core S4 classes, not the wrapper, are 'the primary interface for advanced customization and method development'
- The input is a pathway/feature grid table for 2D multifeature grid heatmaps or list inputs for multi-cohort plots, which fall outside the three-shape dispatch being tested
- Reproducibility auditing of exact package/dependency versions — the article flags that 'Package and dependency version numbers not reported in the document text; session info appears only as unevaluated code'

## Inputs

- Precomputed DE table (data.frame with log2fc, negLog10p, regulation, SampleType, timePoint columns)
- Raw/normalized expression matrix (genes x samples) plus sample metadata data.frame (expr, meta)
- MultiAssayExperiment object (e.g. Bioconductor miniACC example data)
- Package extdata example files from the GitHub repository famanalytics0/MultiModalGraphics

## Outputs

- multimodalplot_routing_results.csv (per-input dispatched class and returned object type)
- S4 visualization objects (ThresholdedScatterplot, AnnotatedHeatmap, CompositeFeatureHeatmap instances)
- Rendered ggplot/ComplexHeatmap figures produced via createPlot() and show()

## How to apply

Load MultiModalGraphics with its dependency stack (MultiAssayExperiment, ComplexHeatmap, ggplot2, dplyr) in an R session. Load the extdata example files from github.com/famanalytics0/MultiModalGraphics covering the three supported input shapes: a raw/normalized expression matrix plus metadata, a precomputed DE table (log2fc, negLog10p, regulation, SampleType, timePoint), and a MultiAssayExperiment object such as miniACC. Exercise the dispatch path — MultiModalPlot() or the explicit classes ThresholdedScatterplot (DE table), ThresholdedScatterplot_table (matrix+metadata via limma/voom), ThresholdedScatterplot_MAE (MAE object) — followed by createPlot() and show() for each input. For each invocation, record which class was dispatched and whether the returned object is an S4 instance or a ggplot, writing per-input outcomes to multimodalplot_routing_results.csv. Judge success by correct class routing per input shape and by plots rendering without the ordering errors the README warns about (e.g. constructing the object before createPlot()).

## Related tools

- **MultiModalGraphics** (Package under test; supplies MultiModalPlot() wrapper, ThresholdedScatterplot/AnnotatedHeatmap/CompositeFeatureHeatmap S4 classes, createPlot(), and extdata example files) — https://github.com/famanalytics0/MultiModalGraphics
- **MultiAssayExperiment** (Provides the MAE object class (e.g. miniACC) used as one of the three dispatch input shapes)
- **ggplot2** (ggplot objects returned by createPlot() are the expected output type recorded by the harness)
- **limma** (Differential expression engine run under the hood when ThresholdedScatterplot_table is dispatched on a matrix+metadata input)

## Examples

```
library(MultiModalGraphics); cs <- ThresholdedScatterplot(df_de, highLog2fc=1.0, lowLog2fc=-1.0, negLog10pValue=1.301); cs <- createPlot(cs); show(cs)
```

## Evaluation signals

- Each extdata input routes to its expected class: raw matrix -> ThresholdedScatterplot path, precomputed DE table -> ThresholdedScatterplot, MAE object -> ThresholdedScatterplot_MAE/CompositeFeatureHeatmap
- Returned object type check: underlying S4 object before createPlot(), ggplot (or ComplexHeatmap) instance after createPlot()
- multimodalplot_routing_results.csv contains one row per extdata input with dispatched class and object type recorded
- show() renders a faceted volcano plot with up/down/neutral colors and per-facet up/down counts for the DE-table input without errors
- Object-construction ordering invariant holds: the S4 object (e.g. cs) exists before createPlot() is called, avoiding the README's documented 'object not found' failure

## Limitations

- The vignette claims to include examples (data generation, scatterplots, informative heatmaps, multifeature grids) but no such example content appears in the document body, so harness expectations must be sourced from the README
- Package and dependency version numbers are not reported in the article text; sessionInfo() appears only as unevaluated code, limiting reproducibility checks
- No named datasets or accession identifiers for the biological/multi-omics data are provided in the vignette; the harness must rely on toy/simulated data and the miniACC Bioconductor example
- Advanced features and customization are deferred to external ComplexHeatmap and ggplot2 documentation and are not covered by this harness

## Evidence

- [readme] extract ggplot or ComplexHeatmap objects for RMarkdown and annotation: [evidence span withheld — anchor status: external_doc]
- [readme] This line must appear first, or you’ll see “object 'cs' not found”: [evidence span withheld — anchor status: external_doc]
