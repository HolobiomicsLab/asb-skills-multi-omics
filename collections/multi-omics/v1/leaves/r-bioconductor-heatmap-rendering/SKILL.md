---
name: r-bioconductor-heatmap-rendering
description: Use when you need to visualize pathway- or feature-level activity matrices
  (fold-change/z-score plus p-value and counts) as an annotated composite heatmap
  in R, e.g. multi-region/timepoint omics summaries.
when_to_use_negative:
- Input is already a rendered figure or a simple one-dimensional expression table
  with no paired statistics (p-values/counts) to overlay — a plain heatmap suffices
- The goal is a volcano plot or differential-expression computation itself rather
  than annotated heatmap rendering of precomputed statistics
- Data are not in R/Bioconductor containers (MAE, matrices, DE tables) and cannot
  be reshaped into fold-change/p-value matrices
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3360
tools:
- name: MultiModalGraphics
  role: Provides CompositeFeatureHeatmap/AnnotatedHeatmap S4 classes and createPlot
    methods that build tile-and-point composite heatmaps
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: ComplexHeatmap
  role: Rendering engine underlying the informative heatmap with overlaid significance
    markers
- name: dplyr
  role: Reshapes the summary table into fold-change and p-value matrices indexed by
    pathway, region, and timepoint
- name: ggplot2
  role: Supports advanced theming and customization of the rendered figure
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:883094ac86a5de5de47f31615b299b815dbc2dc7f174c953771c3004d3735aae
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
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# r-bioconductor-heatmap-rendering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Render publication-quality composite heatmaps in R using MultiModalGraphics and ComplexHeatmap, where tile color encodes fold-change/z-scores, overlaid dot color encodes p-value significance, and dot size encodes feature counts.

## When to use

Use when you need to visualize pathway- or feature-level activity matrices (fold-change/z-score plus p-value and counts) as an annotated composite heatmap in R, e.g. multi-region/timepoint omics summaries.

## When NOT to use

- Input is already a rendered figure or a simple one-dimensional expression table with no paired statistics (p-values/counts) to overlay — a plain heatmap suffices
- The goal is a volcano plot or differential-expression computation itself rather than annotated heatmap rendering of precomputed statistics
- Data are not in R/Bioconductor containers (MAE, matrices, DE tables) and cannot be reshaped into fold-change/p-value matrices

## Inputs

- Pathway/feature summary CSV (e.g. twoD_graphics.csv) with activation z-scores, p-values, and gene counts per feature per condition
- Fold-change (z-score) matrix and matching p-value matrix, optionally with a counts matrix
- Precomputed DE tables, expression/count matrices + metadata, or MultiAssayExperiment objects (per README supported input types)

## Outputs

- Composite feature heatmap PNG/PDF (e.g. fig2_composite_heatmap.png) with z-score tiles, p-value dots, and count-scaled dot sizes
- ComplexHeatmap/ggplot object for further RMarkdown annotation
- sessionInfo() output for reproducibility

## How to apply

Load the summary table (e.g. twoD_graphics.csv from the MultiModalGraphics package extdata via read.csv), then reshape it with dplyr into parallel fold-change (z-score) and p-value matrices indexed by pathway, sample dimension (e.g. brain regions AY, HC, MPFC, SE, ST, VS), and timepoint. Construct a CompositeFeatureHeatmap/AnnotatedHeatmap object with MultiModalGraphics and render it with ComplexHeatmap: map activation z-scores to tile fill, p-values to overlaid dot color, and gene/feature counts to dot size, mirroring the 2D tile-and-point grid the README describes (z-scores as tile fills, p-values as point color, counts as point size). Adjust thresholds and color ramps via parameters such as highLog2fc, lowLog2fc, pvalue_cutoff, and fc_cutoff, and customize with theme_* objects or layer_fun. Export the result with png()/pdf() (e.g. fig2_composite_heatmap.png) and report sessionInfo() for reproducibility. Correct application yields a heatmap where suppressed pathways (negative z-scores) are visually distinguishable from activated ones and significance is legible from dot color/size.

## Related tools

- **MultiModalGraphics** (Provides CompositeFeatureHeatmap/AnnotatedHeatmap S4 classes and createPlot methods that build tile-and-point composite heatmaps) — https://github.com/famanalytics0/MultiModalGraphics
- **ComplexHeatmap** (Rendering engine underlying the informative heatmap with overlaid significance markers)
- **dplyr** (Reshapes the summary table into fold-change and p-value matrices indexed by pathway, region, and timepoint)
- **ggplot2** (Supports advanced theming and customization of the rendered figure)

## Examples

```
library(MultiModalGraphics); twoD <- read.csv(system.file("extdata","twoD_graphics.csv",package="MultiModalGraphics")); png("fig2_composite_heatmap.png"); show(createPlot(CompositeFeatureHeatmap(twoD))); dev.off()
```

## Evaluation signals

- Tile colors correspond to expected z-score signs and magnitudes; suppressed pathways (negative z-scores) such as neurogenesis and synaptic plasticity are visibly distinct from activated ones
- Overlaid dot colors encode p-value significance and dot sizes are proportional to feature/gene counts, matching the three input matrices
- Matrix dimensions match the reshaped data: one row per pathway, columns covering all brain regions (AY, HC, MPFC, SE, ST, VS) and timepoints
- Exported PNG/PDF opens correctly and sessionInfo() is recorded for reproducibility
- No 'object not found' errors: the plot object is constructed (e.g. via the class constructor) before createPlot/show is called

## Limitations

- Advanced features and customization are not documented in the vignette; users are deferred to external ComplexHeatmap and ggplot2 documentation
- Package and dependency version numbers are not reported in the article text; session info appears only as unevaluated code, complicating exact reproduction
- The vignette claims to include examples (data generation, scatterplots, informative heatmaps, multifeature grids) but no such example content appears in the document body; consult the GitHub README instead

## Evidence

- [readme] Thresholds & Colors: All main plotting functions let you set highLog2fc, lowLog2fc, negLog10pValue, pvalue_cutoff, fc_cutoff, and color ramps: [evidence span withheld — anchor status: external_doc]
- [other] shows inhibition of neurogenesis and synaptic plasticity pathways, with tile colors encoding z-scores, dot colors encoding p-value significance, and dot sizes reflecting feature counts.: [evidence span withheld — anchor status: not_found]
