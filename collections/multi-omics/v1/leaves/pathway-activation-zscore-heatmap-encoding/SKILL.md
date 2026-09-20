---
name: pathway-activation-zscore-heatmap-encoding
description: Use when you have a pathway/feature summary table with activation z-scores,
  p-values, and gene counts per condition (e.g., brain region × timepoint) and need
  a single publication-quality heatmap that jointly visualizes all three statistics.
when_to_use_negative:
- Input is already a rendered figure or you only need a plain expression heatmap without
  overlaid statistics — use a standard ComplexHeatmap call instead
- Your data is a raw count matrix without precomputed per-pathway z-scores, p-values,
  and counts — first run the limma/voom differential expression pipeline before this
  visualization step
- You need a volcano plot (log2 fold-change vs. -log10 p-value) rather than a grid
  heatmap — use the ThresholdedScatterplot class instead
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MultiModalGraphics
  role: Provides the AnnotatedHeatmap/CompositeFeatureHeatmap S4 classes and multifeature
    grid plotting for tile-and-point heatmaps
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: ComplexHeatmap
  role: Underlying heatmap engine; layer_fun and theme customization for overlaid
    significance markers
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: dplyr
  role: Reshapes the pathway summary table into fold-change and p-value matrices indexed
    by pathway, region, and timepoint
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: ggplot2
  role: Referenced for advanced features and customization of the composite plots
- name: R
  role: Implementation language; read.csv to load extdata tables, png()/sessionInfo()
    for export and reproducibility
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:104883ad2c027e60a46390d014adaf288d615aa21973f3bcda1eccbd27d79a4a
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# pathway-activation-zscore-heatmap-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Build composite 2D tile-and-point heatmaps with MultiModalGraphics' CompositeFeatureHeatmap/AnnotatedHeatmap classes, encoding pathway activation z-scores as tile fill colors, p-values as overlaid dot color, and gene/feature counts as dot size — applied to pathway summary tables such as the GSE45035 PTSD mouse-model data underlying Fig. 2.

## When to use

Use when you have a pathway/feature summary table with activation z-scores, p-values, and gene counts per condition (e.g., brain region × timepoint) and need a single publication-quality heatmap that jointly visualizes all three statistics.

## When NOT to use

- Input is already a rendered figure or you only need a plain expression heatmap without overlaid statistics — use a standard ComplexHeatmap call instead
- Your data is a raw count matrix without precomputed per-pathway z-scores, p-values, and counts — first run the limma/voom differential expression pipeline before this visualization step
- You need a volcano plot (log2 fold-change vs. -log10 p-value) rather than a grid heatmap — use the ThresholdedScatterplot class instead

## Inputs

- Pathway/feature summary table in CSV (e.g., twoD_graphics.csv from MultiModalGraphics extdata) with columns for pathway, condition (e.g., brain region and timepoint), activation z-score, p-value, and gene/feature count
- Fold-change (z-score) and p-value matrices reshaped from the table via dplyr
- Optionally a MultiAssayExperiment/SummarizedExperiment object or matrix + metadata DE pipeline feeding the FC and p-value matrices

## Outputs

- Composite tile-and-point heatmap (PNG/PDF, e.g., fig2_composite_heatmap.png) with z-scores as tile fill, p-values as dot color, and counts as dot size
- sessionInfo() report for reproducibility

## How to apply

Load the pathway summary table (e.g., twoD_graphics.csv shipped in the MultiModalGraphics package extdata) with read.csv, then reshape it with dplyr into a fold-change (z-score) matrix and a matching p-value matrix indexed by pathway and condition (e.g., brain regions AY, HC, MPFC, SE, ST, VS across timepoints). Load MultiModalGraphics, ComplexHeatmap, and dplyr via suppressPackageStartupMessage(library(...)) as in the vignette. Construct the composite heatmap so tile color maps the z-score, overlaid dot color maps p-value significance, and dot size maps the gene/feature count, consistent with the README's 'multifeature grids' definition. Export the plot with png()/pdf() (e.g., fig2_composite_heatmap.png) and record sessionInfo() for reproducibility. Correctness is judged by tiles correctly encoding signed z-scores, significance dots matching the p-value cutoffs, and suppressed pathways (negative z-scores) being visually distinguishable from activated ones.

## Related tools

- **MultiModalGraphics** (Provides the AnnotatedHeatmap/CompositeFeatureHeatmap S4 classes and multifeature grid plotting for tile-and-point heatmaps) — https://github.com/famanalytics0/MultiModalGraphics
- **ComplexHeatmap** (Underlying heatmap engine; layer_fun and theme customization for overlaid significance markers) — https://github.com/famanalytics0/MultiModalGraphics
- **dplyr** (Reshapes the pathway summary table into fold-change and p-value matrices indexed by pathway, region, and timepoint) — https://github.com/famanalytics0/MultiModalGraphics
- **ggplot2** (Referenced for advanced features and customization of the composite plots)
- **R** (Implementation language; read.csv to load extdata tables, png()/sessionInfo() for export and reproducibility)

## Examples

```
suppressPackageStartupMessage(library(MultiModalGraphics)); suppressPackageStartupMessage(library(ComplexHeatmap)); suppressPackageStartupMessage(library(dplyr))
```

## Evaluation signals

- Tile fill colors correspond one-to-one with signed activation z-scores (negative z-scores for suppressed pathways such as neurogenesis and synaptic plasticity in Fig. 2)
- Overlaid dot colors correctly encode p-value significance relative to the chosen pvalue_cutoff, and dot sizes scale monotonically with feature/gene counts
- The rendered heatmap covers the full condition grid (e.g., five brain regions × four timepoints) without missing tiles
- The exported file (e.g., fig2_composite_heatmap.png) renders and sessionInfo() is reported so the figure is reproducible
- The matrix dimensions of the z-score and p-value inputs match (same pathways × conditions) before plotting

## Limitations

- Advanced features and customization are not documented in the vignette itself; readers are deferred to external ComplexHeatmap and ggplot2 documentation
- Package and dependency version numbers are not reported in the article text (sessionInfo appears only as unevaluated code), so exact reproducibility requires pinning versions separately
- No named datasets or accession identifiers are provided in the vignette body; the GSE45035 provenance comes from the task context rather than the package docs

## Evidence
