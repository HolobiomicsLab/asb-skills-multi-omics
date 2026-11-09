---
name: complexheatmap-legend-assembly
description: Use when a MultiModalGraphics AnnotatedHeatmap must be finalized for
  export and its heatmap-color, CNA/mutation-annotation, and significance-star legends
  need to be packaged into one composite ComplexHeatmap layout.
when_to_use_negative:
- When a quick exploratory look suffices — use the one-line MultiModalPlot() entry
  point instead of assembling custom legends via S4 classes
- When the input is already a single rendered figure and only styling is needed —
  defer to the external ComplexHeatmap/ggplot2 documentation rather than re-deriving
  the annotated-heatmap workflow
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MultiModalGraphics
  role: Provides the AnnotatedHeatmap S4 class whose legends (heatmap colors, annotation
    tracks, star-significance key) are assembled into the composite layout
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: ComplexHeatmap
  role: Rendering backend performing the legend assembly and composite layout of heatmap
    panels and annotation tracks
- name: dplyr
  role: Harmonizes feature identifiers across the mutation/CNA, miRNA-mRNA, DNA methylation,
    and protein-mRNA input tables before the heatmap legends are built
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:d8986b095f0485b636e7a667519f04e0ddcbb08ca9c2efe4051a7bd79cf162a3
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# ComplexHeatmap legend assembly for composite multi-omics figures

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles the legends of a MultiModalGraphics AnnotatedHeatmap — heatmap color ramp, row/column annotation tracks, and star-significance key — into a single composite ComplexHeatmap layout so that publication-quality multi-omics panels (e.g., the six-cancer CESC/OV/PRAD/TGCT/UCEC/UCS figure) render with integrated, self-explanatory keys.

## When to use

Use when a MultiModalGraphics AnnotatedHeatmap must be finalized for export and its heatmap-color, CNA/mutation-annotation, and significance-star legends need to be packaged into one composite ComplexHeatmap layout.

## When NOT to use

- When a quick exploratory look suffices — use the one-line MultiModalPlot() entry point instead of assembling custom legends via S4 classes
- When the input is already a single rendered figure and only styling is needed — defer to the external ComplexHeatmap/ggplot2 documentation rather than re-deriving the annotated-heatmap workflow

## Inputs

- Fold-change matrix and p-value/q-value matrix for features (epigenetic markers, transcripts, proteins)
- Harmonized extdata CSV tables (Pan_cancer_CESC_Mutated_CNV_STR_VART.csv, Pan_cancer_miRNA-mRNA_interaction_data.csv, Pan_Cancer_DNA_methylation.csv, Pan_cancer_protein-mRNA_combined_data.csv)
- CNA/mutation annotation columns for row-annotation tracks
- A MultiModalGraphics AnnotatedHeatmap S4 object

## Outputs

- Composite ComplexHeatmap object with heatmap panels, CNA/mutation annotation tracks, significance-star overlays, and integrated legends
- Rendered figure file, e.g. fig1_annotated_heatmap.png, containing all six cancer panels (CESC, OV, PRAD, TGCT, UCEC, UCS)

## How to apply

After building the AnnotatedHeatmap from fold-change and p-value/q-value matrices (subset here to the six cancer panels CESC, OV, PRAD, TGCT, UCEC, UCS) with significance-star thresholds overlaid on the q-value/p-value values and CNA/mutation columns mapped to row-annotation tracks, collect the individual legends — heatmap colors, annotation-track legends, and the star-significance key — and pass them to ComplexHeatmap's legend-assembly mechanism so they render as a unified composite. Rely on the MultiModalGraphics S4 classes (AnnotatedHeatmap, built on ComplexHeatmap) as the primary interface, since the paper notes core S4 classes remain the interface for advanced customization. Control stringency via the package's threshold parameters (pvalue_cutoff, fc_cutoff, highLog2fc/lowLog2fc) and highlight features via layer_fun if custom star overlays are needed. Export the composed figure by wrapping the plot call in png()/pdf() — here rendering fig1_annotated_heatmap.png — or extract the ComplexHeatmap object directly for RMarkdown embedding.

## Related tools

- **MultiModalGraphics** (Provides the AnnotatedHeatmap S4 class whose legends (heatmap colors, annotation tracks, star-significance key) are assembled into the composite layout) — https://github.com/famanalytics0/MultiModalGraphics
- **ComplexHeatmap** (Rendering backend performing the legend assembly and composite layout of heatmap panels and annotation tracks)
- **dplyr** (Harmonizes feature identifiers across the mutation/CNA, miRNA-mRNA, DNA methylation, and protein-mRNA input tables before the heatmap legends are built)

## Examples

```
suppressPackageStartupMessage(library(MultiModalGraphics)); suppressPackageStartupMessage(library(ComplexHeatmap)); suppressPackageStartupMessage(library(dplyr))
```

## Evaluation signals

- The rendered figure contains all six cancer panels (CESC, OV, PRAD, TGCT, UCEC, UCS) with overlaid significance stars
- CNA/mutation annotation tracks are visible as row-annotation tracks alongside the heatmap body
- Legends for heatmap colors, annotation tracks, and the star-significance key appear as a single integrated composite, not as separate scattered keys
- The exported file (fig1_annotated_heatmap.png) matches the composition of the article's Fig. 1
- Fold-change and q-value/p-value matrices fed to the constructor carry the threshold values (pvalue_cutoff, fc_cutoff) reflected by the plotted star markers

## Limitations

- Advanced features and customization are not documented in the paper's vignette; readers are deferred to the external ComplexHeatmap and ggplot2 documentation
- The vignette claims heatmap/grid examples but no such example content appears in the document body, so legend-assembly details must be confirmed against the package documentation
- Package and dependency version numbers are not reported in the document text; session info appears only as unevaluated code, complicating reproducibility checks

## Evidence

- [discussion] S4 classes as customization interface: "the core S4 classes remain the primary interface for advanced customization and method development"
- [readme] Export guidance: [evidence span withheld — anchor status: external_doc]
- [readme] Significance-marker heatmaps: "**Informative heatmaps** with overlaid significance markers from:"
- [readme] Custom overlays via layer_fun: [evidence span withheld — anchor status: external_doc]
- [abstract] Embedding statistical summaries: "The package allows seamless embedding of statistical summaries such as fold-changes, p-values, q-values, and standard deviations"
