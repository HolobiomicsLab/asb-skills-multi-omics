---
name: gene-count-significance-dot-encoding
description: Use when you have a pathway or feature summary table with activation
  z-scores (fold-changes), p-values, and gene counts across conditions (e.g., brain
  regions and timepoints) and need a single composite heatmap visualizing all three
  modalities.
when_to_use_negative:
- Input is a simple expression/count matrix or precomputed DE table intended for a
  volcano plot or plain heatmap — use ThresholdedScatterplot or AnnotatedHeatmap instead
- The table lacks one of the three required modalities (no p-values or no gene counts),
  since dots encode significance color and count size and cannot be constructed from
  z-scores alone
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0092
tools:
- name: MultiModalGraphics
  role: Provides the CompositeFeatureHeatmap / multifeature-grid 2D tile-and-point
    heatmap functionality encoding z-scores, p-values, and counts
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: ComplexHeatmap
  role: Underlying heatmap engine used with MultiModalGraphics to render informative
    heatmaps with overlaid significance markers
- name: dplyr
  role: Used to reshape the summary table into fold-change and p-value matrices for
    the composite heatmap
- name: ggplot2
  role: Referenced for advanced customization of the composite plot
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:0c33d15b81d23e0df503dc54025907676f343aa8a85e1652d0d0b207e0ffa558
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Gene-count significance dot encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes per-feature (e.g., pathway) gene counts and statistical significance as overlaid dots on a tile heatmap — z-scores as tile fills, p-values as dot color, gene counts as dot size — using MultiModalGraphics' 2D multifeature grid / CompositeFeatureHeatmap built on ComplexHeatmap.

## When to use

Use when you have a pathway or feature summary table with activation z-scores (fold-changes), p-values, and gene counts across conditions (e.g., brain regions and timepoints) and need a single composite heatmap visualizing all three modalities.

## When NOT to use

- Input is a simple expression/count matrix or precomputed DE table intended for a volcano plot or plain heatmap — use ThresholdedScatterplot or AnnotatedHeatmap instead
- The table lacks one of the three required modalities (no p-values or no gene counts), since dots encode significance color and count size and cannot be constructed from z-scores alone

## Inputs

- Pathway/feature grid table (CSV) with per-feature fold-change (z-score), p-value, and gene count per condition (e.g., twoD_graphics.csv from MultiModalGraphics extdata)
- Fold-change (z-score) and p-value matrices indexed by feature and condition
- Pathway-level summaries derived from GSE45035 mouse PTSD model data (five brain regions: AY, HC, MPFC, SE, ST, VS; four timepoints)

## Outputs

- Composite tile-and-point heatmap (e.g., fig2_composite_heatmap.png) with z-score tile colors, p-value dot colors, and gene-count dot sizes
- sessionInfo() report for reproducibility

## How to apply

Load the feature summary table (e.g., twoD_graphics.csv shipped in the MultiModalGraphics package extdata, containing activation z-scores, p-values, and gene counts per pathway per condition) with read.csv, then load MultiModalGraphics, ComplexHeatmap, and dplyr via suppressPackageStartupMessage(library(...)). Reshape the data with dplyr into two aligned matrices indexed by feature and condition: a fold-change (z-score) matrix and a p-value matrix. Build the composite heatmap via MultiModalGraphics/ComplexHeatmap so that tile color encodes activation z-scores, overlaid dot color encodes p-value significance, and dot size encodes gene/feature counts — the package's 'multifeature grid' 2D tile-and-point design. Export the figure with png()/pdf() and report sessionInfo() for reproducibility. Suppressed pathways (negative z-scores) should read as cool-colored tiles with significance dots sized by the number of contributing genes.

## Related tools

- **MultiModalGraphics** (Provides the CompositeFeatureHeatmap / multifeature-grid 2D tile-and-point heatmap functionality encoding z-scores, p-values, and counts) — https://github.com/famanalytics0/MultiModalGraphics
- **ComplexHeatmap** (Underlying heatmap engine used with MultiModalGraphics to render informative heatmaps with overlaid significance markers)
- **dplyr** (Used to reshape the summary table into fold-change and p-value matrices for the composite heatmap)
- **ggplot2** (Referenced for advanced customization of the composite plot)

## Examples

```
suppressPackageStartupMessage(library(MultiModalGraphics)); suppressPackageStartupMessage(library(ComplexHeatmap)); suppressPackageStartupMessage(library(dplyr)); twoD <- read.csv("twoD_graphics.csv")
```

## Evaluation signals

- Tile colors correctly map activation z-scores (negative/suppressed pathways such as neurogenesis and synaptic plasticity in cool colors, positive in warm colors)
- Overlaid dot colors reflect p-value significance and dot sizes are proportional to feature/gene counts per cell
- Matrix dimensions match the feature × condition grid (pathways × brain regions × timepoints, e.g., 6 regions × 4 timepoints for GSE45035)
- Figure exports cleanly to the target PNG (fig2_composite_heatmap.png) and sessionInfo() is reported for reproducibility

## Limitations

- Advanced features and customization are deferred to external ComplexHeatmap and ggplot2 documentation rather than documented in the package guide
- No named datasets or accession identifiers are provided in the vignette text for the biological data demonstrated
- No changelog found and package/dependency version numbers are not reported in the document text

## Evidence

- [intro] Informative heatmaps with significance markers: [evidence span withheld — anchor status: not_found]
